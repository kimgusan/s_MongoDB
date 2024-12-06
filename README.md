# Mongo DB

-   NoSQL : 비관계형 데이터 베이스
-   compass: MongoDB GUI

## collection : 관계형 데이터베이스에서 테이블이라고 이해하면 편함

    1. 전체 데이터베이스 확인
        show dbs
    2. 특정 collection 으로 이동
        use [collection name]

## CRUD

### 1. Create

-   db.[collection name].insertOne() : 데이터 1개 추가
-   db.[collection name].insertMany() : 데이터 n개 추가

tip. new Data() : 현재날짜로 입력

### 2. Read (select)

-   db.[collectionName].find() : 전체 검색
-   db.[collectionName].find(key: value) : 해당 value 값 검색
-   db.[collectionName].find(key: {$gte : 9}) : key 중에서 숫자 9 이상인 값 find  
    ($eq, $gt, $gte, $in, %lt, $lte, $ne, $nin 등)
-   db.movies.find({$or: [{rating: {$gt:9}}, {year: {$gte:2020}}]})
-   db.movies.find({genres: {$size:3}}): 해당 key 중에서 배열 개수가 3개 이상인 값 find

tip. 정규식도 사용 가능하다.

-   db.[collectionName].find({title: {$regex: /the/i}})
-   db.[collectionName].find({director: {$exists: false}}) : 특정 key 가 존재하는 지 확인하는 방법
-   db.[collectionName].find({"cast.0": "Keanu Reeves" }): 특정 key의 n 번째 값을 검색하는 방법
-   db.[collectionName].find().limit(10): 10개만 검색
-   db.[collectionName].find().skip(10): 10개 넘겨서 검색
-   db.[collectionName].find().sort({rating: -1, title:1}).limit(10).skip(10): 특정 key 에 대한 오름차순 내일차순 정렬, 단 점수가 같은경우 알파벳 순으로 하기 위해 title 을 사용

### 3. Update

-   db.movies.updateOne({찾는 데이터}, {변경 데이터})
-   db.movies.updateOne({\_id: ObjectId('')}, {$set: {"director", "kimkyusan"}, $current_date: {updated_at:true}})
-   db.movies.findOneAndUpdate({
    \_id: ObjectId("6752906eec92ffe2b4bd037e")
    },{
    $set: {'director': 'kimyusan'},
$currentDate: {updated_at : true}
    },{returnNewDocument:true, upsert: true}
    ) : 일반 update와의 차이점은 데이터가 없으면 생성하고 있으면 변환 후 반환해라 라는 뜻

-   $inc

    >

          db.movies.updateMany(
          {director: "Christopher Nolan"},
          {
              $inc: {rating:0.2}
          }
          )

-   $push: 추가

    >

        db.movies.updateMany(

        {title: "Inception"},
        {
        $push: {genres:"Mind-Control"}
        }
        ) : update 새로운 내용 추가

-   $pull : 삭제

    >

        db.movies.updateMany(

        {title: "Inception"},
        {
        $pull: {genres:"Mind-Control"}
        }
        )

-   $addToSet : 배열에 존재하지 않는 요소만 추가

    >

        db.movies.updateMany(

        {title: "Inception"},
        {
        $addToSet: {genres:"Mind-Control"}
        }
        )

-   $rename: key 변경

    >

        db.movies.updateMany({}, {$rename:{runtime: "duration"}})

-   $unset: 특정 key 해제 (지우기)

    >

        db.movies.updateMany({}, {$unset: {plot: ""}})

-   $expr: 여러 조건을 중첩해서 검색하는 것 (expression)

    >

        db.movies.updateMany(

        {$expr: {$lt: [{$size: "$genres"}, 3]}},
        {$addToSet: {genres: {$each: ["Other", "Happy"]}}}
        )

### 4. delete

-   db.[collectionName].deleteMany()
-   db.[collectionName].deleteOne({})

## Aggregation (집계함수)

-   count

    >

        db.movies.aggregate([{$count: "total_movies"}])

-   average

    >

        db.movies.aggregate([
            {$group: {_id: null, avgRating:{ $avg:"$rating"}}}
            ])

-   unwind : 배열을 해체

    >

        db.movies.aggregate([
            {$group: {_id: null, avgRating:{ $avg:"$rating"}}}
            ])

    >

        db.movies.aggregate([
        { $unwind: "$genres"},
        {$group: {
            _id: "$genres",
            count: { $sum: 1}

        }},
        {$sort: {count: -1}}
        ])

-   group

        >

            db.movies.aggregate([
            {$group: {
                _id: null,
                oldestMovie: {$min: "$year"},
                newestMovie: {$max: "$year"}
                }
            }
            ]
            )

    >

        db.movies.aggregate([
        {$group: {
        _id: "$year",
        avgDuration: {$avg: "$duration"},
        }
        },
        {$sort: {\_id:-1}}
        ])

    >

        db.movies.aggregate([
        {
            $match: {director: {$exists: true}}
        },
        {$group: {
            _id: "$director",
            movieCount: {$sum: 1},
            }
        },
        {$sort: {movieCount:-1}}
        ])

    >

        db.movies.aggregate(
        {
            $unwind: '$cast'
        },
        {
            $group: {
                _id: "$director",
                movieCount: { $sum: 1}
            }
        },
        {
            $sort: {movieCount:-1}
        }
        )

    >

        db.movies.aggregate([
        {
        $project: {title:1, director:1, cast:1},

        },
        {$limit:10}
        ])
