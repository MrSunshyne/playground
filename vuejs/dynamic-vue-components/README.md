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

The components that are generated are listed in a JSON file in this format :

```json
[
    {
        "name": "Countdown", 
        "path": "Countdown/MyCountdown.cdcbb1c290a530281c17.umd.min.js"
    },
    {
        "name": "Hello", 
        "path" : "MyComponent/MyComponent.df2882f990aacbdc6f27.umd.min.js"
    }
]

```

This json is loaded on `mounted` and then they are pulled using the externalComponent function. 


The magic to import external components is done by using the following function. Yes, this function tries to do what `import` does. Truely magic. 

```js
// src/utils/external-component.js
export default async function externalComponent(url) {
    const name = url.split('/').reverse()[0].match(/^(.*?)\.umd/)[1];
    console.log(name);
  
    if (window[name]) return window[name];
  
    window[name] = new Promise((resolve, reject) => {
      const script = document.createElement('script');
      script.async = true;
      script.addEventListener('load', () => {
        resolve(window[name]);
      });
      script.addEventListener('error', () => {
        reject(new Error(`Error loading ${url}`));
      });
      script.src = url;
      document.head.appendChild(script);
    });
  
    return window[name];
  }
```

```js
  mounted() {
    fetch('/api/component-library.json')
    .then(response => {
      return response.json()
    })
    .then(response => {
      this.library = response.map(res => {
        let result = {
          ...res,
          comp : () => externalComponent(`http://localhost:4444/${res.path}`)
        } 
        return result;
      });
    })
  }
```


Then we iterate over the library data thing to show the external components as follows: 

```html
<div v-if="library">
    <Component v-for="comp in library" :key="comp.name" :is="comp.comp" />
</div>
```