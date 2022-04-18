## How to switch between Angular SSR or CSR based on detecting user-agent

Since SSR is good for SEO and crawler bots, it's not good enough for users. For example, the transitions between pages could be slowed down if your site traffics get high. there are a lot of Pros and Cons for SSR and CSR I'm not digging into them. What we want here is to have both of them at the same time. We want SSR for SEO and CSR for users.

# How

We are going to create an SSR Angular app and enable user-agent detection step by step. Let's Go!

## Create a brand new Angular App

Run the CLI command `ng new my-app` to create a new angular project.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649451382033/lkljcGjSo.png)

## Install Angular SSR

Change the directory to `my-app` and run `ng add @nguniversal/express-engine` 

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650258272122/1TPH2tRb7.png)

## Install the isbot package

In the terminal, run the command `npm install isbot`

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650258533179/2YMCKIG6h.png)

## Config server.ts

In the `server.ts` we going to use isbot to detect whether the user-agent is a bot or not. To do that, we going to import the package

```ts
import isbot from 'isbot';

```
Notice that the isbot is imported as a default import and it's not allowed by default.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650259026202/V73cu2T5B.png)

To allow default import, we set `"allowSyntheticDefaultImports": true,` in `tsconfig.json` to allow default import.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650259072769/QQDC9bTDU.png)

After fixing the default import now it's time to do the magic. We now detect bots and run the application as SSR otherwise run the application as CSR

```ts
// All regular routes use the Universal engine
  server.get('*', (req, res) => {
    if (isbot(req.headers['user-agent'])) {
      console.log('bot and running on SSR');
    res.render(indexHtml, { req, providers: [{ provide: APP_BASE_HREF, useValue: req.baseUrl }] });
    } else {
      console.log('Running on CSR');
      res.sendFile(join(distFolder, `index.html`));
    }
  });
```

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650260169304/CszpW3TPe.png)

## Test

To test how it works we going to build and run the application on SSR and do some checks in the browser.

By clicking on http://localhost:4200 we will see the message in the console log that it's running on CSR.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650260276104/RC0jDps12.png)

Now, we going to check what will happen when the bot is coming to the site. To do that we going to open network conditions in Inspect Element > Newrok > Network Conditions

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650260685188/ELjAe2FzL.png)

Select Googlebot and reload the page:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650260772992/Ol2-fYu4H.png)
to check it by `view on the source` we going to open it and again check it with the default user agent and Googlebot

With the default user agent we will see our application run on CSR:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650261412055/OF0iuA_zf.png)

And with the user-agent as Googlebot, we will see the application in SSR:


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650261486626/sXnbDwJeu.png)

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650261505659/lfj0rpt1B.png)

## Conclusion

With this approach you can serve your heavy application SEO-friendly and User-friendly at the same time. 

Please check out the source code at https://github.com/adnanebrahimi/angular-csr-ssr and don't forget to leave a comment if you like it.

Thanks for reading.


