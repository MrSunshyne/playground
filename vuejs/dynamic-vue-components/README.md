# Building a dynamic vue library

Load vue components from a remote server based on (this article)[https://markus.oberlehner.net/blog/distributed-vue-applications-loading-components-via-http/]


These commands were required before running the build command: 

```
$ npm install @vue/cli-service
$ npm i vue-template-compiler
```

The build command is as follows :

```
npx vue-cli-service build --target lib --formats umd-min --no-clean --dest server/components/MyComponent --name "MyComponent.[chunkhash]" server/components/MyComponent/MyComponent.vue

```

After building the component, serve the js file using http-server -p 4444

