## Build Docker for ExpressJS

- In  expressjs folder, open a terminal and run:

```
docker build -t hellojs .
```
- Run the image:

```
docker run -p 3000:3000 hellojs
```

- Go to web browser and type: http://localhost:3000

## Build Docker for Spring boot
- In springboot folder, open a terminal and run:

```
docker build -t hellospring .
```
- Run the image:

```
docker run -p 8080:8080 hellospring
```
- Go to web browser and type: http://localhost:8080


