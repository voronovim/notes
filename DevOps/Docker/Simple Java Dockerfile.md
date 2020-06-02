# Simple Java Dokerfile

```dockerfile
FROM java:alpine

EXPOSE 8080

COPY ./build/libs/*.jar .

CMD java -jar  *.jar
```

