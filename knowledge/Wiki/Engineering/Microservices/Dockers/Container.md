```bash
docker run -d -p 5000:5000 --restart=always --name registry -v $PWD/registry:/var/lib/registry registry
```

registry is now a image that represent for a docker registry


```shell
docker run -d -p 8080:80 -v $PWD volumn image_name
```



````code


```
