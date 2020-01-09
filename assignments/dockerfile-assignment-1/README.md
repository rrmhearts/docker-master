# Assignment Commentary

Docker file was easy and straight forward to write and build.
```
docker image build -t dockerfile-ass1 .
```

I switch up my internal port and external port! I wrote `3000:80` at first and was so sure it was right. Major fail. 
```
$ docker container run -p 80:3000 --rm dockerfile-ass1
GET / 200 59.946 ms - 290
GET /stylesheets/style.css 200 7.758 ms - 111
GET /images/picard.gif 200 11.630 ms - 417700
```