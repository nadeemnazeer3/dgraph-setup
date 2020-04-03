# What all we need
Check docker-compose.yml
# How to start
```docker-compose up -d```
# Setting up Ratel
```
http://<HOST>:8080/
```
#How to setup on ubuntu:
https://medium.com/@praveenkumarpalai/installing-dgraph-on-ubuntu-the-simple-way-9f1634935a6
# curl cmd to post schema
```
curl -H "Content-Type: application/rdf" -X POST localhost:8080/alter?commitNow=true -d $'
AboutMe: string .
Author: [uid] @reverse .
Owner: [uid] @reverse .
DisplayName: string .
Location: string .
Reputation: int .
Score: int .
Text: string @index(fulltext) .
Tag.Text: string @index(hash) .
Type: string @index(exact) .
ViewCount: int @index(int) .
Vote: [uid] @reverse .
Title: [uid] @reverse .
Body: [uid] @reverse .
Post: [uid] @reverse .
PostCount: int @index(int) .
Tags: [uid] @reverse .
Timestamp: datetime .
GitHubID: string @index(hash) .
Has.Answer: [uid] @reverse @count .
Chosen.Answer: [uid] @count .
Comment: [uid] @reverse .
Upvote: [uid] @reverse .
Downvote: [uid] @reverse .
Tag: [uid] @reverse .
'
```

# Sample Schema
```
<AboutMe>: string .
<Author>: [uid] @reverse .
<Body>: [uid] @reverse .
<Chosen.Answer>: [uid] @count .
<Comment>: [uid] @reverse .
<CreationDate>: default .
<DisplayName>: string .
<Downvote>: [uid] @reverse .
<GitHubID>: string @index(hash) .
<Has.Answer>: [uid] @count @reverse .
<Id>: default .
<LastAccessDate>: default .
<Location>: string .
<Owner>: [uid] @reverse .
<Post>: [uid] @reverse .
<PostCount>: int @index(int) .
<Reputation>: int .
<Score>: int .
<Tag.Text>: string @index(hash) .
<Tag>: [uid] @reverse .
<TagName>: default .
<Tags>: [uid] @reverse .
<Text>: string @index(fulltext) .
<Timestamp>: datetime .
<Title>: [uid] @reverse .
<Type>: string @index(exact) .
<Upvote>: [uid] @reverse .
<ViewCount>: int @index(int) .
<Vote>: [uid] @reverse .
```

# How to use dgraphloader
```
for category in comments posts tags users votes; do docker exec -it dgraph dgraphloader -r $category.rdf.gz; done
```

# How to use dgraph live
```
for category in comments posts tags users votes; do docker exec -it stackof_server_1 dgraph live -f rdf/$category.rdf.gz --alpha 172.25.0.3:9080 --zero 172.25.0.2:5080 -c 1; done
```

# How to use dgraph bulk
```
for category in posts comments tags users votes; do docker exec -it stackof_server_1 dgraph bulk -f rdf/$category.rdf.gz -s so.schema --map_shards=4 --reduce_shards=1 --http localhost:8000 --zero=192.168.48.2:5080; done

docker exec -it stackof_server_1 dgraph bulk -f rdf/posts.rdf.gz -s so.schema --map_shards=4 --reduce_shards=1 --http localhost:8000 --zero=192.168.16.2:5080
```

# A Sample Composite Query
```yaml
{
  
questions(func: eq(Type, "Question"), orderdesc: Timestamp, first: 100) {
  
uid
Id

Title {
  Text
}

Owner {
  DisplayName
  Reputation
  uid
}

Tag {
  TagName: Tag.Text
}

Has.Answer(orderdesc: Timestamp, first: 1) {
  Owner {
    DisplayName
    Reputation
    uid
  }
  Timestamp
}

ChosenAnswerCount: count(Chosen.Answer)
UpvoteCount: count(Upvote)
DownvoteCount: count(Downvote)
AnswerCount: count(Has.Answer)
ViewCount
Timestamp

}

  
t as var(func: eq(Type, "Tag")) {
  c as count(~Tag)
}

topTags(func: uid(t), orderdesc: val(c), first: 10) {
  uid
  TagName: Tag.Text
  QuestionCount: val(c)
}

  
var(func: eq(Type, "Question"), orderdesc: Timestamp, first: 50)  @filter(has(Chosen.Answer)) {
  ca as Chosen.Answer
}

var(func: uid(ca)) @groupby(Owner) {
  a as count(uid)
}

topUsers(func: uid(a), orderdesc: val(a)) {
  uid
  AboutMe
  DisplayName
  Reputation
  NumAcceptedAnswers: val(a)
}

}
```

# Notes
1. You may need to restart dgprah server and/or zero to see the loaded data


