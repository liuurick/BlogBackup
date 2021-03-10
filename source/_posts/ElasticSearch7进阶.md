---
title: ElasticSearch7进阶
date: 2020-12-06 11:10:05
tags: ElasticSearch7
categories: [后端,搜索引擎,ElasticSearch]
---

ElasticSearch的进阶学习主要有两种分别是通读文档和案例学习。这里我选择的是通过案例来深入学习es。

案例数据选择的是tmdb网站的公开数据。

<!--more-->

> tmdb网站：https://www.themoviedb.org/

 # tmdb索引创建

```
PUT /movie
{
   "settings" : {
      "number_of_shards" : 1,
      "number_of_replicas" : 1
   },
   "mappings": {
     "properties": {
       "title":{"type":"text","analyzer": "english"},
       "tagline":{"type":"text","analyzer": "english"},
       "release_date":{"type":"date",        "format": "8yyyy/MM/dd||yyyy/M/dd||yyyy/MM/d||yyyy/M/d"},
       "popularity":{"type":"double"},
       "cast":{
         "type":"object",
         "properties":{
           "character":{"type":"text","analyzer":"standard"},
           "name":{"type":"text","analyzer":"standard"}
         }
       },
       "overview":{"type":"text","analyzer": "english"}
     }
   }
}
```

# tmdb文档导入 

> csv文件导入es用通过SpringBoot实现的：https://github.com/liuurick/csvimportes



# tmdb查询学习

## query DSL

>https://blog.csdn.net/supermao1013/article/details/84261526

### 1.match查询与terms查询

#### match查询

match query 知道分词器的存在，会对field进行分词操作，然后再查询

```
GET /movie/_search
{
  "query": {
    "match": {
      "title": "Steve Jobs"
    }
  }
}
```

结果：

```
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 7,
      "relation" : "eq"
    },
    "max_score" : 14.340551,
    "hits" : [
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "2340",
        "_score" : 14.340551,
        "_source" : {
          "title" : "Steve Jobs",
          "tagline" : "Can a great man be a good man?",
          "release_date" : "2015/10/9",
          "popularity" : "53.670525",
          "cast" : {
            "character" : "Burke",
            "name" : "Aaron Eckhart"
          },
          "overview" : "Set backstage at three iconic product launches and ending in 1998 with the unveiling of the iMac, Steve Jobs takes us behind the scenes of the digital revolution to paint an intimate portrait of the brilliant man at its epicenter."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "703",
        "_score" : 6.936559,
        "_source" : {
          "title" : "The Italian Job",
          "tagline" : "Get in. Get out. Get even.",
          "release_date" : "2003/5/30",
          "popularity" : "62.766854",
          "cast" : {
            "character" : "Jason Bourne",
            "name" : "Matt Damon"
          },
          "overview" : "Charlie Croker pulled off the crime of a lifetime. The one thing that he didn't plan on was being double-crossed. Along with a drop-dead gorgeous safecracker, Croker and his team take off to re-steal the loot and end up in a pulse-pounding, pedal-to-the-metal chase that careens up, down, above and below the streets of Los Angeles."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "1581",
        "_score" : 6.936559,
        "_source" : {
          "title" : "The Nut Job",
          "tagline" : "Let's Get Nuts!",
          "release_date" : "2014/1/17",
          "popularity" : "18.568021",
          "cast" : {
            "character" : "Kay",
            "name" : "Meryl Streep"
          },
          "overview" : "Surly, a curmudgeon, independent squirrel is banished from his park and forced to survive in the city. Lucky for him, he stumbles on the one thing that may be able to save his life, and the rest of park community, as they gear up for winter - Maury's Nut Store."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "2152",
        "_score" : 6.936559,
        "_source" : {
          "title" : "The Bank Job",
          "tagline" : "",
          "release_date" : "2008/2/28",
          "popularity" : "30.387754",
          "cast" : {
            "character" : "Lucius",
            "name" : "Tom Felton"
          },
          "overview" : "Terry is a small-time car dealer trying to leave his shady past behind and start a family. Martine is a beautiful model from Terry's old neighbourhood who knows that Terry is no angel. When Martine proposes a foolproof plan to rob a bank, Terry recognises the danger but realises this may be the opportunity of a lifetime. As the resourceful band of thieves burrows its way into a safe-deposit vault at a Lloyds Bank, they quickly realise that, besides millions in riches, the boxes also contain secrets that implicate everyone from London's most notorious underworld gangsters to powerful government figures, and even the Royal Family. Although the heist makes headlines throughout Britain for several days, a government gag order eventually brings all reporting of the case to an immediate halt."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "4081",
        "_score" : 6.936559,
        "_source" : {
          "title" : "Inside Job",
          "tagline" : "The film that cost $20,000,000,000,000 to make.",
          "release_date" : "2010/10/8",
          "popularity" : "16.930914",
          "cast" : {
            "character" : "",
            "name" : "Adam Beach"
          },
          "overview" : "A film that exposes the shocking truth behind the economic crisis of 2008. The global financial meltdown, at a cost of over $20 trillion, resulted in millions of people losing their homes and jobs. Through extensive research and interviews with major financial insiders, politicians and journalists, Inside Job traces the rise of a rogue industry and unveils the corrosive relationships which have corrupted politics, regulation and academia."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "2552",
        "_score" : 6.198678,
        "_source" : {
          "title" : "All About Steve",
          "tagline" : "A Comedy That Clings",
          "release_date" : "2009/9/4",
          "popularity" : "13.237835",
          "cast" : {
            "character" : "Melvin B. Tolson",
            "name" : "Denzel Washington"
          },
          "overview" : "After one short date, a brilliant crossword constructor decides that a CNN cameraman is her true love. Because the cameraman's job takes him hither and yon, she crisscrosses the country, turning up at media events as she tries to convince him they are perfect for each other."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "979",
        "_score" : 5.3308554,
        "_source" : {
          "title" : "The Life Aquatic with Steve Zissou",
          "tagline" : "The deeper you go, the weirder life gets.",
          "release_date" : "2004/12/10",
          "popularity" : "25.237969",
          "cast" : {
            "character" : "Hannibal Lecter",
            "name" : "Gaspard Ulliel"
          },
          "overview" : "Wes Anderson��s incisive quirky comedy build up stars complex characters like in ��The Royal Tenenbaums�� with Bill Murray on in the leading role. An ocean adventure documentary film maker Zissou is put in all imaginable life situations and a tough life crisis as he attempts to make a new film about capturing the creature that caused him pain."
        }
      }
    ]
  }
}

```



#### 查看分词分析

```
GET /movie/_analyze
{
  "field": "title", 
  "text": "Steve Jobs"
}
```

通过下方结果可知"Steve Jobs"被分词为steve和job

```
{
  "tokens" : [
    {
      "token" : "steve",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "job",
      "start_offset" : 6,
      "end_offset" : 10,
      "type" : "<ALPHANUM>",
      "position" : 1
    }
  ]
}
```



#### term查询

term：查询某个字段里含有某个关键词的文档

terms：查询某个字段里含有多个关键词的文档

```

GET /movie/_search
{
  "query": {
    "terms": {
      "title": [
        "Steve Jobs"
      ]
    }
  }
}
```

通过下方结果可知并没有命中：

```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}
```

#### 总结

match查询是按照字段上的定义的分词分析后去索引内查询

term查询是不进行分词的查询，直接去索引内查询，这种查询适合keyword、numeric、date等明确值的



### 2.match分词后的and和or

match查询默认是or

```
GET /movie/_search
{
 "query":{
  "match":{"title":"basketball with cartoom aliens"}
 }
}
```

查询结果：

```
{
  "took" : 12,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 11,
      "relation" : "eq"
    },
    "max_score" : 8.280251,
    "hits" : [
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "2550",
        "_score" : 8.280251,
        "_source" : {
          "title" : "Love & Basketball",
          "tagline" : "All's fair in love and basketball.",
          "release_date" : "2000/4/21",
          "popularity" : "2.027393",
          "cast" : {
            "character" : "Laurie Strode",
            "name" : "Jamie Lee Curtis"
          },
          "overview" : "A young African-American couple navigates the tricky paths of romance and athletics in this drama. Quincy McCall (Omar Epps) and Monica Wright (Sanaa Lathan) grew up in the same neighborhood and have known each other since childhood. As they grow into adulthood, they fall in love, but they also share another all-consuming passion: basketball. They've followed the game all their lives and have no small amount of talent on the court. As Quincy and Monica struggle to make their relationship work, they follow separate career paths though high school and college basketball and, they hope, into stardom in big-league professional ball."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "839",
        "_score" : 7.7807794,
        "_source" : {
          "title" : "Alien³",
          "tagline" : "The bitch is back.",
          "release_date" : "1992/5/22",
          "popularity" : "45.856409",
          "cast" : {
            "character" : "Beatrix 'The Bride' Kiddo",
            "name" : "Uma Thurman"
          },
          "overview" : "After escaping with Newt and Hicks from the alien planet, Ripley crash lands on Fiorina 161, a prison planet and host to a correctional facility. Unfortunately, although Newt and Hicks do not survive the crash, a more unwelcome visitor does. The prison does not allow weapons of any kind, and with aid being a long time away, the prisoners must simply survive in any way they can."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "2389",
        "_score" : 7.7807794,
        "_source" : {
          "title" : "Aliens",
          "tagline" : "This Time It's War",
          "release_date" : "1986/7/18",
          "popularity" : "67.66094",
          "cast" : {
            "character" : "Conner",
            "name" : "Jason Momoa"
          },
          "overview" : "When Ripley's lifepod is found by a salvage crew over 50 years later, she finds that terra-formers are on the very planet they found the alien species. When the company sends a family of colonists out to investigate her story, all contact is lost with the planet and colonists. They enlist Ripley and the colonial marines to return and search for answers."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "3144",
        "_score" : 7.7807794,
        "_source" : {
          "title" : "Alien",
          "tagline" : "In space no one can hear you scream.",
          "release_date" : "1979/5/25",
          "popularity" : "94.184658",
          "cast" : {
            "character" : "Raimunda",
            "name" : "Penu00e9lope Cruz"
          },
          "overview" : "During its return to the earth, commercial spaceship Nostromo intercepts a distress signal from a distant planet. When a three-member team of the crew discovers a chamber containing thousands of eggs on the planet, a creature inside one of the eggs attacks an explorer. The entire crew is unaware of the impending nightmare set to descend upon them when the alien parasite planted inside its unfortunate host is birthed."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "741",
        "_score" : 6.26783,
        "_source" : {
          "title" : "Alien: Resurrection",
          "tagline" : "It's already too late.",
          "release_date" : "1997/11/12",
          "popularity" : "37.44963",
          "cast" : {
            "character" : "Jennings",
            "name" : "Ben Affleck"
          },
          "overview" : "Two hundred years after Lt. Ripley died, a group of scientists clone her, hoping to breed the ultimate weapon. But the new Ripley is full of surprises �� as are the new aliens. Ripley must team with a band of smugglers to keep the creatures from reaching Earth."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "1087",
        "_score" : 6.26783,
        "_source" : {
          "title" : "Aliens in the Attic",
          "tagline" : "The aliens vs. the Pearsons",
          "release_date" : "2009/7/31",
          "popularity" : "13.707183",
          "cast" : {
            "character" : "Hova",
            "name" : "Julia Roberts"
          },
          "overview" : "It's summer vacation, but the Pearson family kids are stuck at a boring lake house with their nerdy parents. That is until feisty, little, green aliens crash-land on the roof, with plans to conquer the house AND Earth! Using only their wits, courage and video game-playing skills, the youngsters must band together to defeat the aliens and save the world - but the toughest part might be keeping the whole thing a secret from their parents! Featuring an all-star cast including Ashley Tisdale, Andy Richter, Kevin Nealon, Tim Meadows and Doris Roberts, Aliens In The Attic is the most fun you can have on this planet!"
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "3347",
        "_score" : 6.26783,
        "_source" : {
          "title" : "Alien Zone",
          "tagline" : "Don't you dare go in there!",
          "release_date" : "1978/11/22",
          "popularity" : "0.000372",
          "cast" : {
            "character" : "Sgt. Jericho 'Action' Jackson",
            "name" : "Carl Weathers"
          },
          "overview" : "A man who is having an affair with a married woman is dropped off on the wrong street when going back to his hotel. He takes refuge out of the rain when an old man invites him in. He turns out to be a mortician, who tells him the stories of the people who have wound up in his establishment over the course of four stories."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "68",
        "_score" : 5.2474747,
        "_source" : {
          "title" : "Monsters vs Aliens",
          "tagline" : "When aliens attack, monsters fight back.",
          "release_date" : "2009/3/19",
          "popularity" : "36.167578",
          "cast" : {
            "character" : "Carl Fredricksen (voice)",
            "name" : "Ed Asner"
          },
          "overview" : "When Susan Murphy is unwittingly clobbered by a meteor full of outer space gunk on her wedding day, she mysteriously grows to 49-feet-11-inches. The military jumps into action and captures Susan, secreting her away to a covert government compound. She is renamed Ginormica and placed in confinement with a ragtag group of Monsters..."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "2468",
        "_score" : 5.2474747,
        "_source" : {
          "title" : "My Stepmother is an Alien",
          "tagline" : "Man's closest encounter.",
          "release_date" : "1988/12/9",
          "popularity" : "9.455596",
          "cast" : {
            "character" : "Cristina",
            "name" : "Scarlett Johansson"
          },
          "overview" : "Trying to rescue her home planet from destruction, a gorgeous extraterrestrial named Celeste arrives on Earth and begins her scientific research. She woos quirky scientist Dr. Steve Mills, a widower with a young daughter. Before long, Celeste finds herself in love with Steve and her new life on Earth, where she experiences true intimacy for the first time. But when she loses sight of her mission, she begins to question where she belongs."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "1214",
        "_score" : 4.512821,
        "_source" : {
          "title" : "Aliens vs Predator: Requiem",
          "tagline" : "The Last Place On Earth We Want To Be Is In The Middle",
          "release_date" : "2007/12/25",
          "popularity" : "39.381913",
          "cast" : {
            "character" : "Alex Wyler",
            "name" : "Keanu Reeves"
          },
          "overview" : "A sequel to 2004's Alien vs. Predator, the iconic creatures from two of the scariest film franchises in movie history wage their most brutal battle ever - in our own backyard. The small town of Gunnison, Colorado becomes a war zone between two of the deadliest extra-terrestrial life forms - the Alien and the Predator. When a Predator scout ship crash-lands in the hills outside the town, Alien Facehuggers and a hybrid Alien/Predator are released and begin to terrorize the town."
        }
      }
    ]
  }
}
```

将or换成and，想同时匹配到basketball 和cartoom aliens

```
GET /movie/_search
{
 "query":{
  "match": {
   "title": {
     "query": "basketball with cartoom aliens",
     "operator": "and" 
   }
  }
 } 
}
```

查询结果：

 ```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}
 ```

### 3.最小词项匹配

```
GET /movie/_search
{
 "query":{
  "match": {
   "title": {
     "query": "basketball with cartoom aliens",
     "operator": "or" ,
     "minimum_should_match": 2
   }
  }
 }
}
```



```
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}
```



### 4.短语查询

```
GET /movie/_search
{
 "query":{
  "match_phrase":{"title":"steve zissou"}
 }
}
```

查询结果：

```
{
  "took" : 13,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 11.292614,
    "hits" : [
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "979",
        "_score" : 11.292614,
        "_source" : {
          "title" : "The Life Aquatic with Steve Zissou",
          "tagline" : "The deeper you go, the weirder life gets.",
          "release_date" : "2004/12/10",
          "popularity" : "25.237969",
          "cast" : {
            "character" : "Hannibal Lecter",
            "name" : "Gaspard Ulliel"
          },
          "overview" : "Wes Anderson��s incisive quirky comedy build up stars complex characters like in ��The Royal Tenenbaums�� with Bill Murray on in the leading role. An ocean adventure documentary film maker Zissou is put in all imaginable life situations and a tough life crisis as he attempts to make a new film about capturing the creature that caused him pain."
        }
      }
    ]
  }
}

```



**短语前缀查询:**

```
GET /movie/_search
{
 "query":{
  "match_phrase_prefix":{"title":"steve zis"}
 }
}
```

查询结果：

```
{
  "took" : 14,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 23.216133,
    "hits" : [
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "979",
        "_score" : 23.216133,
        "_source" : {
          "title" : "The Life Aquatic with Steve Zissou",
          "tagline" : "The deeper you go, the weirder life gets.",
          "release_date" : "2004/12/10",
          "popularity" : "25.237969",
          "cast" : {
            "character" : "Hannibal Lecter",
            "name" : "Gaspard Ulliel"
          },
          "overview" : "Wes Anderson��s incisive quirky comedy build up stars complex characters like in ��The Royal Tenenbaums�� with Bill Murray on in the leading role. An ocean adventure documentary film maker Zissou is put in all imaginable life situations and a tough life crisis as he attempts to make a new film about capturing the creature that caused him pain."
        }
      }
    ]
  }
}
```



### 5.多字段查询

```
GET /movie/_search
{
 "query":{
  "multi_match":{
   "query":"basketball with cartoom aliens",
   "field":["title","overview"]
  }
 }
}
```

查询结果：

```
{
  "error" : {
    "root_cause" : [
      {
        "type" : "parsing_exception",
        "reason" : "[multi_match] unknown token [START_ARRAY] after [field]",
        "line" : 5,
        "col" : 12
      }
    ],
    "type" : "parsing_exception",
    "reason" : "[multi_match] unknown token [START_ARRAY] after [field]",
    "line" : 5,
    "col" : 12
  },
  "status" : 400
}
```



操作不管是字符与还是或，按照逻辑关系命中后相加得分

> "explain": true 类似于MySQL中的explain 

```
GET /movie/_search
{
 "explain": true, 
 "query":{
  "match":{"title":"steve"}
 }
}
```



 查询结果:

```
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 7.4039927,
    "hits" : [
      {
        "_shard" : "[movie][0]",
        "_node" : "NpA3du0ARfyfdH55kWTlrw",
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "2340",
        "_score" : 7.4039927,
        "_source" : {
          "title" : "Steve Jobs",
          "tagline" : "Can a great man be a good man?",
          "release_date" : "2015/10/9",
          "popularity" : "53.670525",
          "cast" : {
            "character" : "Burke",
            "name" : "Aaron Eckhart"
          },
          "overview" : "Set backstage at three iconic product launches and ending in 1998 with the unveiling of the iMac, Steve Jobs takes us behind the scenes of the digital revolution to paint an intimate portrait of the brilliant man at its epicenter."
        },
        "_explanation" : {
          "value" : 7.4039927,
          "description" : "weight(title:steve in 1437) [PerFieldSimilarity], result of:",
          "details" : [
            {
              "value" : 7.4039927,
              "description" : "score(freq=1.0), computed as boost * idf * tf from:",
              "details" : [
                {
                  "value" : 2.2,
                  "description" : "boost",
                  "details" : [ ]
                },
                {
                  "value" : 7.1592917,
                  "description" : "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
                  "details" : [
                    {
                      "value" : 3,
                      "description" : "n, number of documents containing term",
                      "details" : [ ]
                    },
                    {
                      "value" : 4500,
                      "description" : "N, total number of documents with field",
                      "details" : [ ]
                    }
                  ]
                },
                {
                  "value" : 0.47008157,
                  "description" : "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
                  "details" : [
                    {
                      "value" : 1.0,
                      "description" : "freq, occurrences of term within document",
                      "details" : [ ]
                    },
                    {
                      "value" : 1.2,
                      "description" : "k1, term saturation parameter",
                      "details" : [ ]
                    },
                    {
                      "value" : 0.75,
                      "description" : "b, length normalization parameter",
                      "details" : [ ]
                    },
                    {
                      "value" : 2.0,
                      "description" : "dl, length of field",
                      "details" : [ ]
                    },
                    {
                      "value" : 2.1757777,
                      "description" : "avgdl, average length of field",
                      "details" : [ ]
                    }
                  ]
                }
              ]
            }
          ]
        }
      },
      {
        "_shard" : "[movie][0]",
        "_node" : "NpA3du0ARfyfdH55kWTlrw",
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "2552",
        "_score" : 6.198678,
        "_source" : {
          "title" : "All About Steve",
          "tagline" : "A Comedy That Clings",
          "release_date" : "2009/9/4",
          "popularity" : "13.237835",
          "cast" : {
            "character" : "Melvin B. Tolson",
            "name" : "Denzel Washington"
          },
          "overview" : "After one short date, a brilliant crossword constructor decides that a CNN cameraman is her true love. Because the cameraman's job takes him hither and yon, she crisscrosses the country, turning up at media events as she tries to convince him they are perfect for each other."
        },
        "_explanation" : {
          "value" : 6.198678,
          "description" : "weight(title:steve in 1632) [PerFieldSimilarity], result of:",
          "details" : [
            {
              "value" : 6.198678,
              "description" : "score(freq=1.0), computed as boost * idf * tf from:",
              "details" : [
                {
                  "value" : 2.2,
                  "description" : "boost",
                  "details" : [ ]
                },
                {
                  "value" : 7.1592917,
                  "description" : "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
                  "details" : [
                    {
                      "value" : 3,
                      "description" : "n, number of documents containing term",
                      "details" : [ ]
                    },
                    {
                      "value" : 4500,
                      "description" : "N, total number of documents with field",
                      "details" : [ ]
                    }
                  ]
                },
                {
                  "value" : 0.39355576,
                  "description" : "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
                  "details" : [
                    {
                      "value" : 1.0,
                      "description" : "freq, occurrences of term within document",
                      "details" : [ ]
                    },
                    {
                      "value" : 1.2,
                      "description" : "k1, term saturation parameter",
                      "details" : [ ]
                    },
                    {
                      "value" : 0.75,
                      "description" : "b, length normalization parameter",
                      "details" : [ ]
                    },
                    {
                      "value" : 3.0,
                      "description" : "dl, length of field",
                      "details" : [ ]
                    },
                    {
                      "value" : 2.1757777,
                      "description" : "avgdl, average length of field",
                      "details" : [ ]
                    }
                  ]
                }
              ]
            }
          ]
        }
      },
      {
        "_shard" : "[movie][0]",
        "_node" : "NpA3du0ARfyfdH55kWTlrw",
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "979",
        "_score" : 5.3308554,
        "_source" : {
          "title" : "The Life Aquatic with Steve Zissou",
          "tagline" : "The deeper you go, the weirder life gets.",
          "release_date" : "2004/12/10",
          "popularity" : "25.237969",
          "cast" : {
            "character" : "Hannibal Lecter",
            "name" : "Gaspard Ulliel"
          },
          "overview" : "Wes Anderson��s incisive quirky comedy build up stars complex characters like in ��The Royal Tenenbaums�� with Bill Murray on in the leading role. An ocean adventure documentary film maker Zissou is put in all imaginable life situations and a tough life crisis as he attempts to make a new film about capturing the creature that caused him pain."
        },
        "_explanation" : {
          "value" : 5.3308554,
          "description" : "weight(title:steve in 140) [PerFieldSimilarity], result of:",
          "details" : [
            {
              "value" : 5.3308554,
              "description" : "score(freq=1.0), computed as boost * idf * tf from:",
              "details" : [
                {
                  "value" : 2.2,
                  "description" : "boost",
                  "details" : [ ]
                },
                {
                  "value" : 7.1592917,
                  "description" : "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
                  "details" : [
                    {
                      "value" : 3,
                      "description" : "n, number of documents containing term",
                      "details" : [ ]
                    },
                    {
                      "value" : 4500,
                      "description" : "N, total number of documents with field",
                      "details" : [ ]
                    }
                  ]
                },
                {
                  "value" : 0.33845747,
                  "description" : "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
                  "details" : [
                    {
                      "value" : 1.0,
                      "description" : "freq, occurrences of term within document",
                      "details" : [ ]
                    },
                    {
                      "value" : 1.2,
                      "description" : "k1, term saturation parameter",
                      "details" : [ ]
                    },
                    {
                      "value" : 0.75,
                      "description" : "b, length normalization parameter",
                      "details" : [ ]
                    },
                    {
                      "value" : 4.0,
                      "description" : "dl, length of field",
                      "details" : [ ]
                    },
                    {
                      "value" : 2.1757777,
                      "description" : "avgdl, average length of field",
                      "details" : [ ]
                    }
                  ]
                }
              ]
            }
          ]
        }
      }
    ]
  }
}

```

查看数值，tfidf多少分，tfnorm归一化后多少分

![image-20201206201041743](/images/2020120601.png)

多字段查询索引内有query分词后的结果，因为title比overview命中更重要，因此需要加权重。

### 6.多字段查询优化

```
GET /movie/_search
{
 "query":{
  "multi_match":{
   "query":"basketball with cartoom aliens",
   "fields":["title^10","overview"],
   "tie_break":0.3
  }
 }
}
```

查询结果：

```
{
  "error" : {
    "root_cause" : [
      {
        "type" : "parsing_exception",
        "reason" : "[multi_match] query does not support [tie_break]",
        "line" : 6,
        "col" : 16
      }
    ],
    "type" : "parsing_exception",
    "reason" : "[multi_match] query does not support [tie_break]",
    "line" : 6,
    "col" : 16
  },
  "status" : 400
}
```



## 继续深入查询

### 1.bool查询

>must：必须都是true
>
>must not： 必须都是false
>
>should：其中有一个为true即可，但true的越多得分越高



```
GET /movie/_search
{
 "query":{
  "bool": { 
   "should": [
    { "match": { "title":"basketball with cartoom aliens"}}, 

    { "match": { "overview":"basketball with cartoom aliens"}} 
   ]
  }
 }
}
```



查询结果：

```
{
  "took" : 23,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 103,
      "relation" : "eq"
    },
    "max_score" : 14.092542,
    "hits" : [
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "2550",
        "_score" : 14.092542,
        "_source" : {
          "title" : "Love & Basketball",
          "tagline" : "All's fair in love and basketball.",
          "release_date" : "2000/4/21",
          "popularity" : "2.027393",
          "cast" : {
            "character" : "Laurie Strode",
            "name" : "Jamie Lee Curtis"
          },
          "overview" : "A young African-American couple navigates the tricky paths of romance and athletics in this drama. Quincy McCall (Omar Epps) and Monica Wright (Sanaa Lathan) grew up in the same neighborhood and have known each other since childhood. As they grow into adulthood, they fall in love, but they also share another all-consuming passion: basketball. They've followed the game all their lives and have no small amount of talent on the court. As Quincy and Monica struggle to make their relationship work, they follow separate career paths though high school and college basketball and, they hope, into stardom in big-league professional ball."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "2389",
        "_score" : 11.687689,
        "_source" : {
          "title" : "Aliens",
          "tagline" : "This Time It's War",
          "release_date" : "1986/7/18",
          "popularity" : "67.66094",
          "cast" : {
            "character" : "Conner",
            "name" : "Jason Momoa"
          },
          "overview" : "When Ripley's lifepod is found by a salvage crew over 50 years later, she finds that terra-formers are on the very planet they found the alien species. When the company sends a family of colonists out to investigate her story, all contact is lost with the planet and colonists. They enlist Ripley and the colonial marines to return and search for answers."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "1087",
        "_score" : 11.652669,
        "_source" : {
          "title" : "Aliens in the Attic",
          "tagline" : "The aliens vs. the Pearsons",
          "release_date" : "2009/7/31",
          "popularity" : "13.707183",
          "cast" : {
            "character" : "Hova",
            "name" : "Julia Roberts"
          },
          "overview" : "It's summer vacation, but the Pearson family kids are stuck at a boring lake house with their nerdy parents. That is until feisty, little, green aliens crash-land on the roof, with plans to conquer the house AND Earth! Using only their wits, courage and video game-playing skills, the youngsters must band together to defeat the aliens and save the world - but the toughest part might be keeping the whole thing a secret from their parents! Featuring an all-star cast including Ashley Tisdale, Andy Richter, Kevin Nealon, Tim Meadows and Doris Roberts, Aliens In The Attic is the most fun you can have on this planet!"
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "839",
        "_score" : 11.450155,
        "_source" : {
          "title" : "Alien³",
          "tagline" : "The bitch is back.",
          "release_date" : "1992/5/22",
          "popularity" : "45.856409",
          "cast" : {
            "character" : "Beatrix 'The Bride' Kiddo",
            "name" : "Uma Thurman"
          },
          "overview" : "After escaping with Newt and Hicks from the alien planet, Ripley crash lands on Fiorina 161, a prison planet and host to a correctional facility. Unfortunately, although Newt and Hicks do not survive the crash, a more unwelcome visitor does. The prison does not allow weapons of any kind, and with aid being a long time away, the prisoners must simply survive in any way they can."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "3144",
        "_score" : 11.37727,
        "_source" : {
          "title" : "Alien",
          "tagline" : "In space no one can hear you scream.",
          "release_date" : "1979/5/25",
          "popularity" : "94.184658",
          "cast" : {
            "character" : "Raimunda",
            "name" : "Penu00e9lope Cruz"
          },
          "overview" : "During its return to the earth, commercial spaceship Nostromo intercepts a distress signal from a distant planet. When a three-member team of the crew discovers a chamber containing thousands of eggs on the planet, a creature inside one of the eggs attacks an explorer. The entire crew is unaware of the impending nightmare set to descend upon them when the alien parasite planted inside its unfortunate host is birthed."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "1214",
        "_score" : 10.803101,
        "_source" : {
          "title" : "Aliens vs Predator: Requiem",
          "tagline" : "The Last Place On Earth We Want To Be Is In The Middle",
          "release_date" : "2007/12/25",
          "popularity" : "39.381913",
          "cast" : {
            "character" : "Alex Wyler",
            "name" : "Keanu Reeves"
          },
          "overview" : "A sequel to 2004's Alien vs. Predator, the iconic creatures from two of the scariest film franchises in movie history wage their most brutal battle ever - in our own backyard. The small town of Gunnison, Colorado becomes a war zone between two of the deadliest extra-terrestrial life forms - the Alien and the Predator. When a Predator scout ship crash-lands in the hills outside the town, Alien Facehuggers and a hybrid Alien/Predator are released and begin to terrorize the town."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "741",
        "_score" : 10.5949,
        "_source" : {
          "title" : "Alien: Resurrection",
          "tagline" : "It's already too late.",
          "release_date" : "1997/11/12",
          "popularity" : "37.44963",
          "cast" : {
            "character" : "Jennings",
            "name" : "Ben Affleck"
          },
          "overview" : "Two hundred years after Lt. Ripley died, a group of scientists clone her, hoping to breed the ultimate weapon. But the new Ripley is full of surprises �� as are the new aliens. Ripley must team with a band of smugglers to keep the creatures from reaching Earth."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "1740",
        "_score" : 9.010006,
        "_source" : {
          "title" : "How to Lose Friends & Alienate People",
          "tagline" : "He's across the pond, and out of his depth.",
          "release_date" : "2008/10/2",
          "popularity" : "16.785866",
          "cast" : {
            "character" : "Narrator",
            "name" : "Philippe Labro"
          },
          "overview" : """A British writer struggles to fit in at a high-profile magazine in New York. Based on Toby Young's memoir "How to Lose Friends &amp; Alienate People"."""
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "453",
        "_score" : 8.579647,
        "_source" : {
          "title" : "Space Jam",
          "tagline" : "Get ready to jam.",
          "release_date" : "1996/11/15",
          "popularity" : "36.125715",
          "cast" : {
            "character" : "Cameron Poe",
            "name" : "Nicolas Cage"
          },
          "overview" : "In a desperate attempt to win a basketball match and earn their freedom, the Looney Tunes seek the aid of retired basketball champion, Michael Jordan."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "2842",
        "_score" : 7.242802,
        "_source" : {
          "title" : "Just Wright",
          "tagline" : "In this game every shot counts.",
          "release_date" : "2010/5/14",
          "popularity" : "10.409253",
          "cast" : {
            "character" : "Quentin Jacobsen",
            "name" : "Nat Wolff"
          },
          "overview" : "A physical therapist falls for the basketball player she is helping recover from a career-threatening injury."
        }
      }
    ]
  }
}
```

 

 

### 2.不同的multi_query的type

和multi_match得分不一样

因为multi_match有很多种type

best_fields:默认，取得分最高的作为对应的分数，最匹配模式,等同于dismax模式

```
GET /movie/_search

{
 "query":{
  "dis_max": { 
   "queries": [
    { "match": { "title":"basketball with cartoom aliens"}}, 
    { "match": { "overview":"basketball with cartoom aliens"}} 
   ]
  }
 }
}
```



查询结果：

```
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4512,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "title" : "Avatar",
          "tagline" : "Enter the World of Pandora.",
          "release_date" : "2009/12/10",
          "popularity" : "150.437577",
          "cast" : {
            "character" : "Jake Sully",
            "name" : "Sam Worthington"
          },
          "overview" : "In the 22nd century, a paraplegic Marine is dispatched to the moon Pandora on a unique mission, but becomes torn between following orders and protecting an alien civilization."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "title" : "Spectre",
          "tagline" : "A Plan No One Escapes",
          "release_date" : "2015/10/26",
          "popularity" : "107.376788",
          "cast" : {
            "character" : "James Bond",
            "name" : "Daniel Craig"
          },
          "overview" : "A cryptic message from Bond��s past sends him on a trail to uncover a sinister organization. While M battles political forces to keep the secret service alive, Bond peels back the layers of deceit to reveal the terrible truth behind SPECTRE."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : 1.0,
        "_source" : {
          "title" : "The Dark Knight Rises",
          "tagline" : "The Legend Ends",
          "release_date" : "2012/7/16",
          "popularity" : "112.31295",
          "cast" : {
            "character" : "Bruce Wayne / Batman",
            "name" : "Christian Bale"
          },
          "overview" : "Following the death of District Attorney Harvey Dent, Batman assumes responsibility for Dent's crimes to protect the late attorney's reputation and is subsequently hunted by the Gotham City Police Department. Eight years later, Batman encounters the mysterious Selina Kyle and the villainous Bane, a new terrorist leader who overwhelms Gotham's finest. The Dark Knight resurfaces to protect a city that has branded him an enemy."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "5",
        "_score" : 1.0,
        "_source" : {
          "title" : "John Carter",
          "tagline" : "Lost in our world, found in another.",
          "release_date" : "2012/3/7",
          "popularity" : "43.926995",
          "cast" : {
            "character" : "John Carter",
            "name" : "Taylor Kitsch"
          },
          "overview" : "John Carter is a war-weary, former military captain who's inexplicably transported to the mysterious and exotic planet of Barsoom (Mars) and reluctantly becomes embroiled in an epic conflict. It's a world on the brink of collapse, and Carter rediscovers his humanity when he realizes the survival of Barsoom and its people rests in his hands."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "6",
        "_score" : 1.0,
        "_source" : {
          "title" : "Spider-Man 3",
          "tagline" : "The battle within.",
          "release_date" : "2007/5/1",
          "popularity" : "115.699814",
          "cast" : {
            "character" : "Peter Parker / Spider-Man",
            "name" : "Tobey Maguire"
          },
          "overview" : "The seemingly invincible Spider-Man goes up against an all-new crop of villain �� including the shape-shifting Sandman. While Spider-Man��s superpowers are altered by an alien organism, his alter ego, Peter Parker, deals with nemesis Eddie Brock and also gets caught up in a love triangle."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "7",
        "_score" : 1.0,
        "_source" : {
          "title" : "Tangled",
          "tagline" : "They're taking adventure to new lengths.",
          "release_date" : "2010/11/24",
          "popularity" : "48.681969",
          "cast" : {
            "character" : "Flynn Rider (voice)",
            "name" : "Zachary Levi"
          },
          "overview" : "When the kingdom's most wanted-and most charming-bandit Flynn Rider hides out in a mysterious tower, he's taken hostage by Rapunzel, a beautiful and feisty tower-bound teen with 70 feet of magical, golden hair. Flynn's curious captor, who's looking for her ticket out of the tower where she's been locked away for years, strikes a deal with the handsome thief and the unlikely duo sets off on an action-packed escapade, complete with a super-cop horse, an over-protective chameleon and a gruff gang of pub thugs."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "8",
        "_score" : 1.0,
        "_source" : {
          "title" : "Avengers: Age of Ultron",
          "tagline" : "A New Age Has Come.",
          "release_date" : "2015/4/22",
          "popularity" : "134.279229",
          "cast" : {
            "character" : "Tony Stark / Iron Man",
            "name" : "Robert Downey Jr."
          },
          "overview" : "When Tony Stark tries to jumpstart a dormant peacekeeping program, things go awry and Earth��s Mightiest Heroes are put to the ultimate test as the fate of the planet hangs in the balance. As the villainous Ultron emerges, it is up to The Avengers to stop him from enacting his terrible plans, and soon uneasy alliances and unexpected action pave the way for an epic and unique global adventure."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "9",
        "_score" : 1.0,
        "_source" : {
          "title" : "Harry Potter and the Half-Blood Prince",
          "tagline" : "Dark Secrets Revealed",
          "release_date" : "2009/7/7",
          "popularity" : "98.885637",
          "cast" : {
            "character" : "Harry Potter",
            "name" : "Daniel Radcliffe"
          },
          "overview" : "As Harry begins his sixth year at Hogwarts, he discovers an old book marked as 'Property of the Half-Blood Prince', and begins to learn more about Lord Voldemort's dark past."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "10",
        "_score" : 1.0,
        "_source" : {
          "title" : "Batman v Superman: Dawn of Justice",
          "tagline" : "Justice or revenge",
          "release_date" : "2016/3/23",
          "popularity" : "155.790452",
          "cast" : {
            "character" : "Bruce Wayne / Batman",
            "name" : "Ben Affleck"
          },
          "overview" : "Fearing the actions of a god-like Super Hero left unchecked, Gotham City��s own formidable, forceful vigilante takes on Metropolis��s most revered, modern-day savior, while the world wrestles with what sort of hero it really needs. And with Batman and Superman at war with one another, a new threat quickly arises, putting mankind in greater danger than it��s ever known before."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "11",
        "_score" : 1.0,
        "_source" : {
          "title" : "Superman Returns",
          "tagline" : "",
          "release_date" : "2006/6/28",
          "popularity" : "57.925623",
          "cast" : {
            "character" : "Superman / Clark Kent",
            "name" : "Brandon Routh"
          },
          "overview" : "Superman returns to discover his 5-year absence has allowed Lex Luthor to walk free, and that those he was closest too felt abandoned and have moved on. Luthor plots his ultimate revenge that could see millions killed and change the face of the planet forever, as well as ridding himself of the Man of Steel."
        }
      }
    ]
  }
}
```



然后使用explan看下 ((title:steve title:job) | (overview:steve overview:job))，打分规则

```
GET /movie/_validate/query?explain
{
 "query":{
  "multi_match":{
   "query":"steve job",
   "fields":["title","overview"],
   "operator": "or",
   "type":"best_fields"
  }
 }
}
```



查询结果：

```
{
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "valid" : true,
  "explanations" : [
    {
      "index" : "movie",
      "valid" : true,
      "explanation" : "((overview:steve overview:job) | (title:steve title:job))"
    }
  ]
}
```



以字段为单位分别计算分词的分数，然后取最好的一个,适用于最优字段匹配。

 

 

将其他因素以0.3的倍数考虑进去

```
GET /movie/_search
{
 "query":{
  "dis_max": { 
   "queries": [
    { "match": { "title":"basketball with cartoom aliens"}}, 
    { "match": { "overview":"basketball with cartoom aliens"}} 
   ],
   "tie_breaker": 0.3
  }
 }
}
```



 查询结果：

```
{
  "took" : 10,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 103,
      "relation" : "eq"
    },
    "max_score" : 10.023938,
    "hits" : [
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "2550",
        "_score" : 10.023938,
        "_source" : {
          "title" : "Love & Basketball",
          "tagline" : "All's fair in love and basketball.",
          "release_date" : "2000/4/21",
          "popularity" : "2.027393",
          "cast" : {
            "character" : "Laurie Strode",
            "name" : "Jamie Lee Curtis"
          },
          "overview" : "A young African-American couple navigates the tricky paths of romance and athletics in this drama. Quincy McCall (Omar Epps) and Monica Wright (Sanaa Lathan) grew up in the same neighborhood and have known each other since childhood. As they grow into adulthood, they fall in love, but they also share another all-consuming passion: basketball. They've followed the game all their lives and have no small amount of talent on the court. As Quincy and Monica struggle to make their relationship work, they follow separate career paths though high school and college basketball and, they hope, into stardom in big-league professional ball."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "2389",
        "_score" : 8.952852,
        "_source" : {
          "title" : "Aliens",
          "tagline" : "This Time It's War",
          "release_date" : "1986/7/18",
          "popularity" : "67.66094",
          "cast" : {
            "character" : "Conner",
            "name" : "Jason Momoa"
          },
          "overview" : "When Ripley's lifepod is found by a salvage crew over 50 years later, she finds that terra-formers are on the very planet they found the alien species. When the company sends a family of colonists out to investigate her story, all contact is lost with the planet and colonists. They enlist Ripley and the colonial marines to return and search for answers."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "839",
        "_score" : 8.881593,
        "_source" : {
          "title" : "Alien³",
          "tagline" : "The bitch is back.",
          "release_date" : "1992/5/22",
          "popularity" : "45.856409",
          "cast" : {
            "character" : "Beatrix 'The Bride' Kiddo",
            "name" : "Uma Thurman"
          },
          "overview" : "After escaping with Newt and Hicks from the alien planet, Ripley crash lands on Fiorina 161, a prison planet and host to a correctional facility. Unfortunately, although Newt and Hicks do not survive the crash, a more unwelcome visitor does. The prison does not allow weapons of any kind, and with aid being a long time away, the prisoners must simply survive in any way they can."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "3144",
        "_score" : 8.859726,
        "_source" : {
          "title" : "Alien",
          "tagline" : "In space no one can hear you scream.",
          "release_date" : "1979/5/25",
          "popularity" : "94.184658",
          "cast" : {
            "character" : "Raimunda",
            "name" : "Penu00e9lope Cruz"
          },
          "overview" : "During its return to the earth, commercial spaceship Nostromo intercepts a distress signal from a distant planet. When a three-member team of the crew discovers a chamber containing thousands of eggs on the planet, a creature inside one of the eggs attacks an explorer. The entire crew is unaware of the impending nightmare set to descend upon them when the alien parasite planted inside its unfortunate host is birthed."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "453",
        "_score" : 8.579647,
        "_source" : {
          "title" : "Space Jam",
          "tagline" : "Get ready to jam.",
          "release_date" : "1996/11/15",
          "popularity" : "36.125715",
          "cast" : {
            "character" : "Cameron Poe",
            "name" : "Nicolas Cage"
          },
          "overview" : "In a desperate attempt to win a basketball match and earn their freedom, the Looney Tunes seek the aid of retired basketball champion, Michael Jordan."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "1087",
        "_score" : 7.8832817,
        "_source" : {
          "title" : "Aliens in the Attic",
          "tagline" : "The aliens vs. the Pearsons",
          "release_date" : "2009/7/31",
          "popularity" : "13.707183",
          "cast" : {
            "character" : "Hova",
            "name" : "Julia Roberts"
          },
          "overview" : "It's summer vacation, but the Pearson family kids are stuck at a boring lake house with their nerdy parents. That is until feisty, little, green aliens crash-land on the roof, with plans to conquer the house AND Earth! Using only their wits, courage and video game-playing skills, the youngsters must band together to defeat the aliens and save the world - but the toughest part might be keeping the whole thing a secret from their parents! Featuring an all-star cast including Ashley Tisdale, Andy Richter, Kevin Nealon, Tim Meadows and Doris Roberts, Aliens In The Attic is the most fun you can have on this planet!"
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "1214",
        "_score" : 7.644126,
        "_source" : {
          "title" : "Aliens vs Predator: Requiem",
          "tagline" : "The Last Place On Earth We Want To Be Is In The Middle",
          "release_date" : "2007/12/25",
          "popularity" : "39.381913",
          "cast" : {
            "character" : "Alex Wyler",
            "name" : "Keanu Reeves"
          },
          "overview" : "A sequel to 2004's Alien vs. Predator, the iconic creatures from two of the scariest film franchises in movie history wage their most brutal battle ever - in our own backyard. The small town of Gunnison, Colorado becomes a war zone between two of the deadliest extra-terrestrial life forms - the Alien and the Predator. When a Predator scout ship crash-lands in the hills outside the town, Alien Facehuggers and a hybrid Alien/Predator are released and begin to terrorize the town."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "741",
        "_score" : 7.5659513,
        "_source" : {
          "title" : "Alien: Resurrection",
          "tagline" : "It's already too late.",
          "release_date" : "1997/11/12",
          "popularity" : "37.44963",
          "cast" : {
            "character" : "Jennings",
            "name" : "Ben Affleck"
          },
          "overview" : "Two hundred years after Lt. Ripley died, a group of scientists clone her, hoping to breed the ultimate weapon. But the new Ripley is full of surprises �� as are the new aliens. Ripley must team with a band of smugglers to keep the creatures from reaching Earth."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "2842",
        "_score" : 7.242802,
        "_source" : {
          "title" : "Just Wright",
          "tagline" : "In this game every shot counts.",
          "release_date" : "2010/5/14",
          "popularity" : "10.409253",
          "cast" : {
            "character" : "Quentin Jacobsen",
            "name" : "Nat Wolff"
          },
          "overview" : "A physical therapist falls for the basketball player she is helping recover from a career-threatening injury."
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "1890",
        "_score" : 6.92179,
        "_source" : {
          "title" : "He Got Game",
          "tagline" : "The father, the son and the holy game.",
          "release_date" : "1998/5/1",
          "popularity" : "10.232599",
          "cast" : {
            "character" : "Megan",
            "name" : "Shoshana Bush"
          },
          "overview" : "A basketball player's father must try to convince him to go to a college so he can get a shorter sentence."
        }
      }
    ]
  }
}
```

most_fields:取命中的分值相加作为分数，同should match模式，加权共同影响模式

 

然后使用explain看下 ((title:steve title:job) | (overview:steve overview:job))~1.0，打分规则

```
GET /movie/_validate/query?explain
{
 "query":{
  "multi_match":{
   "query":"steve job",
   "fields":["title","overview"],
   "operator": "or",
   "type":"most_fields"
  }
 }
}
```



查询结果：

```
{
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "valid" : true,
  "explanations" : [
    {
      "index" : "movie",
      "valid" : true,
      "explanation" : "((overview:steve overview:job) | (title:steve title:job))~1.0"
    }
  ]
}
```



以字段为单位分别计算分词的分数，然后加在一起，适用于都有影响的匹配

cross_fields:以分词为单位计算栏位总分

然后使用explain看下 blended(terms:[title:steve, overview:steve]) blended(terms:[title:job, overview:job])，打分规则

```
GET /movie/_validate/query?explain
{
 //"explain": true, 
 "query":{
  "multi_match":{
   "query":"steve job",
   "fields":["title","overview"],
   "operator": "or",
   "type":"most_fields"
  }
 }
}
```



以词为单位，分别用词去不同的字段内取内容，拿高的分数后与其他词的分数相加，适用于词导向的匹配

```
GET /forum/article/_search
{
 "query": {
  "multi_match": {
   "query": "Peter Smith",
   "type": "cross_fields", 
   "operator": "or",
   "fields": ["author_first_name", "author_last_name"]
  }
 }
}
```



> 要求Peter必须在author_first_name或author_last_name中出现
>
> 要求Smith必须在author_first_name或author_last_name中出
>
> 原来most_fiels，可能像Smith //Williams也可能会出现，因为most_fields要求只是任何一个field匹配了就可以，匹配的field越多，分数越高

 ```
GET /movie/_search
{
 "explain": true, 
 "query":{
  "multi_match":{
   "query":"steve job",
   "fields":["title","overview"],
   "operator": "or",
   "type":"cross_fields"
  }
 }
}
 ```

看一下不同的评分规则

### 3.query string

方便的利用AND(+) OR(|) NOT(-)

```
GET /movie/_search
{
 "query":{
  "query_string":{
   "fields":["title"],
   "query":"steve AND jobs"
  }
 }
}
```



### 4.过滤查询

filter过滤查询

**单条件过滤**

```
GET /movie/_search
{
 "query":{
  "bool":{
   "filter":{
     "term":{"title":"steve"}
   }
  }
 }
}
```



**多条件过滤**

```
GET /movie/_search
{
 "query":{
  "bool":{
   "filter":[
     {"term":{"title":"steve"}},
     {"term":{"cast.name":"gaspard"}},
     {"range": { "release_date": { "lte": "2015/01/01" }}},
     {"range": { "popularity": { "gte": "25" }}}
     ]
  }
 },
 "sort":[
  {"popularity":{"order":"desc"}}
 ]
}
```



带match打分的的filter

```
GET /movie/_search
{
 "query":{
  "bool":{
   "must": [
     { "match": { "title":  "Search"    }}, 
     { "match": { "tagline": "Elasticsearch" }} 
   ],
   "filter":[
     {"term":{"title":"steve"}},
     {"term":{"cast.name":"gaspard"}},
     {"range": { "release_date": { "lte": "2015/01/01" }}},
     {"range": { "popularity": { "gte": "25" }}}
     ]
  }
 }
}
```

返回0结果

```
GET /movie/_search
{
 "query":{
  "bool":{
   "should": [
     { "match": { "title":  "Search"    }}, 
     { "match": { "tagline": "Elasticsearch" }} 
   ],
   "filter":[
     {"term":{"title":"steve"}},
     {"term":{"cast.name":"gaspard"}},
     {"range": { "release_date": { "lte": "2015/01/01" }}},
     {"range": { "popularity": { "gte": "25" }}}
     ]
  }
 }
}
```

 

有结果，但是返回score为0，因为bool中若有filter的话，即便should都不满足，只是返回为0分而已

修改为

```
GET /movie/_search
{
 "query":{
  "bool":{
   "should": [
     { "match": { "title":  "life"    }}, 
     { "match": { "tagline": "Elasticsearch" }} 
   ],
   "filter":[
     {"term":{"title":"steve"}},
     {"term":{"cast.name":"gaspard"}},
     {"range": { "release_date": { "lte": "2015/01/01" }}},
     {"range": { "popularity": { "gte": "25" }}}
     ]
  }
 }
}
```

可以看到分数



# 查全率和查准率

> 查全率：正确的结果有n个，实际查询出来正确的有m  **查全率：**m/n
>
> 查准率：查出的n个文档有m个正确  **查准率：m/n**
>
> 两者不可兼得，但可以调整排序

function score自定义打分

 ```
GET /movie/_search
{
 "query":{
  "function_score": {
   //原始查询得到oldscore
   "query": {   
     "multi_match":{
     "query":"steve job",
     "fields":["title","overview"],
     "operator": "or",
     "type":"most_fields"
   }
  },
  "functions": [
   {"field_value_factor": {
      "field": "popularity",  //对应要处理的字段
      "modifier": "log2p",  //将字段值+2后，计算对数
      "factor": 10  //字段预处理*10
     }
   }
  ], 
 
  "score_mode": "sum",  //不同的field value之间的得分相加
  "boost_mode": "sum"  //最后在与old value相加
 }
}
}
 ```

# 相关性排序调整

> https://www.cnblogs.com/orzlin/p/10496869.html