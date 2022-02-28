# Embargoed Cloudflare Worker
I use Cloudflare workers for everything I can, and wanted a way for folks to easily add the [Embargoed](https://github.com/rameerez/embargoed-list) response to their sites regardless of having to add a gem, making sure the response doesn't get cached, etc.

Since Cloudflare workers are just a small collection of code, I'll share the steps for set up.

## 1. Create the Worker
Create a service and name it something nice:

<img width="835" alt="Adding a Cloudflare Worker" src="https://user-images.githubusercontent.com/2278221/155936357-c4a858a8-5ae0-4a0f-a781-53048a515945.png">
(you can pick whatever starter you want since we're just going to replace the code)

## 2. Edit the Worker
Edit the Worker's code via the "Quick Edit" function and replace whatever is in there with the following:
```javascript
async function handleRequest(request) {
// HTML from https://github.com/rameerez/embargoed/blob/main/public/maintenance.html
let html = `<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Page blocked for 🇷🇺 Russian visitors</title>

    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.6.0/dist/css/bootstrap.min.css" integrity="sha384-B0vP5xmATw1+K9KRQjQERJvTumQW0nPEzvF6L/Z6nronJ3oUOFUFpCjEUQouq2+l" crossorigin="anonymous">


    <style type="text/css">

        *{
            font-family: Lato, Arial, Helvetica, sans-serif;
        }

        body{
            margin: 0;
            background-color: #f6f9fc;
            height: 100vh;
        }

        .card{
            position: relative;
            padding: 3em 3em;
            text-align: center;
            background: #ffffff;
            color: rgb(66, 84, 102);
            box-shadow: rgba(50, 50, 93, 0.25) 0px 50px 100px -20px, rgba(0, 0, 0, 0.3) 0px 30px 60px -30px;
            border-radius: 8px;
        }

        h1{
            margin: 0;
            color: #242424;
            font-size: 26pt;
            margin-bottom: 1em;
            font-weight: bolder;
        }

    </style>

</head>
<body>

<div class="container h-100">
    <div class="row align-items-center h-100">
        <div class="col-12 col-md-8 offset-md-1 col-lg-6 offset-lg-3 mx-auto">
            <div class="card">
                <h1>We're sorry, Russians.</h1>

                <p>We've blocked this website for Russian visitors.</p>
                <p>On Feb 24, 2022 Russian forces started invading Ukraine. We know you, a regular person, are not to blame. But your leaders have chosen a very violent path breaking international law, and we've chosen not to do business with Russian citizens at the time. We condemn this crime.</p>
                <p>We don't like this situation either, and we'd love to do business with Russia again soon. But only you can make it stop.</p>
                <p><b>Protest in social media and in the streets</b> to demand your leaders stop this invasion. Peace and diplomacy are the only way forward.</p>
                <p>🇺🇦 Slava Ukraini</p>
            </div>
        </div>
    </div>
</div>

</body>
</html>`;

  // Return the HTML if the visitor is in RU
  if (request.cf.country === 'RU') {
    return new Response(html, {
      headers: {
        'content-type': 'text/html;charset=UTF-8',
      },
    });
  }

  // Else, return the page normally :)
  return fetch(request);
}

addEventListener("fetch", event => {
  event.respondWith(handleRequest(event.request));
});
```

Now save and deploy your worker **to production**. Don't worry if you test the worker in the little Quick Edit and it gives you a 500... unless you have a VPN set to put you in RU, it wouldn't work.

## 3. Apply your Worker
Go to the site that you would like the notice to show up on, and click "Workers". Add a route to one page, or all pages, like what I've done below:

<img width="835" alt="Adding a Cloudflare Worker" src="https://user-images.githubusercontent.com/2278221/155938401-a48f56c7-afa6-45c9-9e5e-e1b86349174d.png">

Hit Save! All routes will now display the [Embargoed message](https://raw.githubusercontent.com/rameerez/embargoed/main/public/embargoed-message.jpg) if the visitor's country is RU.

The benefit to the Cloudflare Worker approach is Russian traffic won't ever hit your origin server, and if you have multiple domains in your Cloudflare account, you can apply the Worker very easily to each site without having to make a Worker per-site.

Remember, Cloudflare Workers are not fully free, so if you have have a high-traffic site, you might have to pay a small fee.

## 4. To Test
If you want to test this functionality yourself since potentially you could totally break everything on your site if you didn't do this correctly, or if my code sucks, you should limit your route (test with `example.com/testing123` as the route in the Cloudflare worker first) AND change the country code to one you can test in a VPN. For example, I changed the country code to `MX` (for Mexico) and set my VPN (Nord VPN) to put me in Mexico. Make sure to test in an Incognito window, since I found that worked better for me.

## 5. Disclaimer
**I tested this code to the best of my ability, but please do your own testing and consider the implications of activating this on your site. I take no responsibility for any issues that you have with this code.**
