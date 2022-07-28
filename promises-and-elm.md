---
title: "Promises and Elm"
tags: Elm
date: 2021-02-09
---

For pole-prediction I'm experimenting with writing server-side Elm code. Alex Korban's [Elm-weekly](https://www.elmweekly.nl/) newsletter alerted me to [a nice article](https://dev.to/eberfreitas/elm-in-the-server-or-anywhere-else-with-promises-5eoj?utm_campaign=Elm%20Weekly&utm_medium=email&utm_source=Revue%20newsletter) which suggests using Javascript promises. The stated benefit of this is that it:

> decouples the Elm code and ports from Express itself

I'm not sure I understand the benefit of using Promises here. So I'm going to describe in a bit more detail the current solution in pole-prediction which doesn't use promises and then re-show Eber Freitas Dias's promise-using code. 


Here is the gist of what I currently have on the Javascript side of the server code. It's not exactly this because it contains a few more uninteresting details, I've whittled it down here for the main idea:

```javascript
const app = http.createServer((request, response) => {
    let data = '';
      request.on('data', chunk => {
        data += chunk;
      })
      request.on('end', () => {
        elmApp.ports.requestPort.send({ request: request, body: data, response : response });
      })

});
```

So the `data` stuff is just getting the body of the request such that we can give it to the Elm code. The important part here is that we have to send the `response` into the Elm code. On the Elm side, we never decode the `response` value from an `Encode.Value`, we just pass it around so that we can then send it back. Here is the Javascript code for dealing with the port the Elm code uses to send the `response` back:

```javascript
elmApp.ports.responsePort.subscribe(function(message) {
    var response = message.response;
    var content_type = message.content_type;
    var contents = message.contents;
    response.writeHead(message.response_code, { "Content-Type": content_type});
    // So this is basically assuming the content type is "application/json"
    response.write(JSON.stringify(contents));
    response.end();
});
```

So you see the `response` is sent into the Elm code and just passed straight back out unmodified. Okay, all fine. Let's see what the proposed code with promises is, this is reproduced from the [linked to article](https://dev.to/eberfreitas/elm-in-the-server-or-anywhere-else-with-promises-5eoj?utm_campaign=Elm%20Weekly&utm_medium=email&utm_source=Revue%20newsletter):

```elm
http
  .createServer((request, res) => {
    new Promise(resolve => app.ports.onRequest.send({ request, resolve }))
      .then(({ status, response }) => {
        res.statusCode = status;
        res.end(response);
      });
  })
  .listen(3000);

app.ports.resolve.subscribe(([{ resolve }, status, response]) => {
  resolve({ status, response });
});

```

So my issue here is that you still need to set up the `resolve` port. Elm still needs to accept an `Encode.Value`  in through the `onRequest` port and send that back through the `resolve` port. In this case the code has been whittled down more significantly so that they do not bother about the possibility of sending back a JSON body. But still, you still have two ports, you still pass into Elm an `Encode.Value` which is passed around within Elm and used to make the response which still happens via the `resolve` port. My `requestPort` is essentially the same as the `onRequest` port and my `responsePort` is essentially the same as the `resolve` port. So I completely fail to understand what benefit using the promise has brought?

I guess it means that the `resolve` port is **always** the same even if we moved this code to operate in some other scenario. So suppose you were building something else you could use the same Elm code. However, as you can see in my example, the Elm code is expecting that you are going to interpret the `message.contents` so it's a little hard to see how my Elm code would be immediately possible in some other scenario. In other words, the Elm code is already coupled somewhat. I suppose this might make testing a little easier, but if you're going to take the 'Express' server out of the equation for testing purposes, just test the Elm code directly.

So all-in-all, it's a nice article but I'm left a little confused as to the proposed benefits of using promises. To put a final way, what can I achieve here with promises that I couldn't achieve without the promises?


