# ribanez
My personal website

This project was generated with [Hugo](https://gohugo.io).

### Local development
```
> hugo serve
```

### Deploying
```
> rm -rf public
> hugo server -D --renderToDisk --baseURL=https://ribanez.com.ar --appendPort=false
> hugo deploy --target=ribanez
> aws cloudfront create-invalidation --distribution-id E118KAQ7JBX258 --paths '/*'
```
Or using npm
```
> npm run deploy
```

