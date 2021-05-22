---
title: 抵御跨站脚本（XSS）攻击
date: 2021-05-17 17:49:23
tags:
---

代码实现XSS防御

<!--more-->

# 1 什么是XSS？

- XSS攻击通常指的是利用网页开发时留下的漏洞，通过巧妙地方法注入恶意指令代码到网页，使用户加载并执行攻击者恶意制造的网页程序。这些恶意网页程序通常是js，但实际上也可以包括Java，VBScript，ActiveX，Flash或者甚至是普通的HTML。攻击成功后，攻击者可能得到包括但不限于更高的权限（如执行一些操作），私密网页内容，会话和cookie等各种内容。
- 例如用户在发帖或者注册的时候，在文本框中输入`<script>alert('xss')</script>`，这段代码如果不经过转义处理，而直接保存到数据库。将来视图层渲染HTML的时候，把这段代码输出到页面上，那么`<script>`标签的内容就会被执行。
- 通常情况下，我们登录到某个网站。如果网站使用`HttpSession`保存登录凭证，那么`SessionId`会以`Cookie`的形式保存在游览器上。如果黑客在这个网页发帖的时候，填写的js代码是用来获取cookie内容的，并且把cookie的内容通过ajax发送给黑客的服务器，那么黑客就可以伪造登录。
- 即便很多网站使用了JWT，登录凭证（Token令牌）是存储在游览器上面的。所以用XSS脚本可以轻松的从Storage中提取Token，黑客依然可以轻松的冒充已经登录的用户。
- 所以避免XSS攻击最有效的方法就是对用户输入的数据进行转义，然后存储到数据库里面。等到视图层渲染HTML页面的时候。转义后的文字是不会被当做js执行的，这就可以抵御XSS攻击。

2、代码
问题？
如何去转义用户输入信息
过滤器，可以继承于HttpServletRequest的接口
但是由于这个接口定义的方法太多，一一实现比较麻烦，因此我们有更好的选择
更好的选择
自定义请求类的方式
继承HttpServletRequestWrapper父类
JavaEE规范还定义 了 HttpServletRequestWrapper ，这个类是请求类的包装类，用上了装饰器模式。不得不说这里 用到的设计模式真的非常棒，无论各家应用服务器厂商怎么去实现 HttpServletRequest 接口， 用户想要自定义请求，只需要继承 HttpServletRequestWrapper ，对应覆盖某个方法即可，然后 把请求传入请求包装类，装饰器模式就会替代请求对象中对应的某个方法。用户的代码和服务器 厂商的代码完全解耦，我们不用关心 HttpServletRequest 接口是怎么实现的，借助于包装类我 们可以随意修改请求中的方法。

上代码
public class XssHttpServletRequestWrapper extends HttpServletRequestWrapper {


    public XssHttpServletRequestWrapper(HttpServletRequest request) {
       super(request);
    }
    
    @Override
    public String getParameter(String name) {
        String value = super.getParameter(name);
        if(!StrUtil.hasEmpty(value)){
            value = HtmlUtil.filter(value);
        }
        return value;
    }
    
    @Override
    public String[] getParameterValues(String name) {
        String[] values = super.getParameterValues(name);
        if(ArrayUtil.isNotEmpty(values)){
            for (int i = 0; i < values.length; i++) {
                String value =values[i];
                if(!StrUtil.hasEmpty(values[i])){
                    value = HtmlUtil.filter(value);
                }
                values[i] = value;
            }
        }
        return values;
    }
    
    @Override
    public Map<String, String[]> getParameterMap() {
    
        Map<String, String[]> parameterMap = super.getParameterMap();
        LinkedHashMap<String, String[]> map = new LinkedHashMap<>();
        if(CollUtil.isNotEmpty(parameterMap)){
            for (String key: parameterMap.keySet()) {
                String[] values = parameterMap.get(key);
                    if(ArrayUtil.isNotEmpty(values)){
                        for (int i = 0; i < values.length; i++) {
                            String value =values[i];
                            if(!StrUtil.hasEmpty(values[i])){
                                value = HtmlUtil.filter(value);
                            }
                            values[i] = value;
                        }
                    }
                    map.put(key,values);
            }
        }
        return map;
    }
    
    @Override
    public String getHeader(String name) {
        String header = super.getHeader(name);
        if(StrUtil.isNotBlank(header)){
           header = HtmlUtil.filter(header);
        }
        return header;
    }
      @Override
    public ServletInputStream getInputStream() throws IOException {
        InputStream in = super.getInputStream();
        StringBuffer body = new StringBuffer();
        InputStreamReader reader = new InputStreamReader(in, Charset.forName("UTF-8"));
        BufferedReader buffer = new BufferedReader(reader);
        String line = buffer.readLine();
        while (line != null) {
            body.append(line);
            line = buffer.readLine();
        }
        buffer.close();
        reader.close();
        in.close();
        Map<String, Object> map = JSONUtil.parseObj(body.toString());
        Map<String, Object> resultMap = new HashMap(map.size());
        for (String key : map.keySet()) {
            Object val = map.get(key);
            if (map.get(key) instanceof String) {
                resultMap.put(key, HtmlUtil.filter(val.toString()));
            } else {
                resultMap.put(key, val);
            }
        }
        String str = JSONUtil.toJsonStr(resultMap);
        final ByteArrayInputStream bain = new ByteArrayInputStream(str.getBytes());
        return new ServletInputStream() {
            @Override
            public int read() throws IOException {
                return bain.read();
            }
    
            @Override
            public boolean isFinished() {
                return false;
            }
    
            @Override
            public boolean isReady() {
                return false;
            }
    
            @Override
            public void setReadListener(ReadListener listener) {
            }
        };
    }
}


自定义过滤器
为了让刚刚定义的包装类生效，我们还要在 com.example.emos.wx.config.xss 中创建 XssFilter 过滤器。过滤器拦截所有请求，然后把请求传入包装类，这样包装类就能覆盖所有请 求的参数方法，用户从请求中获得数据，全都经过转义。

@WebFilter(urlPatterns = "/*")  //过滤所有请求路径
public class XssFilter implements Filter {


    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    
    }
    
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse,
                         FilterChain filterChain) throws IOException, ServletException {
        XssHttpServletRequestWrapper servletRequestWrapper = new XssHttpServletRequestWrapper(
            (HttpServletRequest) servletRequest);
        filterChain.doFilter(servletRequestWrapper,servletResponse);
    
    }
    
    @Override
    public void destroy() {
    
    }
}

给主启动类添加@ServletComponentScan注解

