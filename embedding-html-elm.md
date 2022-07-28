---
title: "Embedding HTML into an Elm program"
tags: programming elm
date: 2021-02-15
---

Someone asked on the Elm discourse [how to embed HTML into an Elm app](https://discourse.elm-lang.org/t/how-do-i-embed-long-text-documents-into-a-spa/6914). There are basically two approaches, which I will try to describe along with advantages and disadvantages. 

## Translate the HTML to Elm

Either by hand, or you can use [an online tool](https://mbylstra.github.io/html-to-elm/). As the original poster seemed to find, there is a limit to how sophisticated such a tool is, but it's probably worth a shot. However, this is only going to work if the HTML you are trying to display is not going to change. In the discourse posts, the HTML was what was presented to them from a legal website drafting a set of Terms and Conditions. But if say that HTML was being produced by some wysiwyg HTML editor, then unless you can automate the translation of the HTML to Elm this isn't going to work. It obviously will definitely not be able to display *dynamically generated* HTML.

## Use an HTML-displaying Element

The other strategy is to use an element within Elm that is itself capable of displaying aribitrary HTML. The advantages of these three approaches is that they are pretty universal in that they will display pretty much any HTML that you throw at it. If you update the HTML, you don't really have to change anything to get updated output in your Elm app. The disadvantage is that all three solutions are a little out of the *'Elm philosophy'*. You're essentially cheating the shadow-dom. This can also lead to some slightly strange behaviour, but for the most part these solutions will just work.

Another significant disadvantage of these approaches is that they represent a potential security issue, because you're basically allowing the display of arbitrary HTML (which could, for example, include code to read from your local storage and send that somewhere). If you entirely control this arbitrary HTML, say for example it is  a hard-coded string in your Elm app source code, then this is okay, but if you're using these techniques to display something that some random user might write in, then you're doing something wrong.

### Use a Custom Element

Custom elements are something of an escape hatch in Elm. Normally, once you're in Elm, you're in the land of non-stateful updates etc. However, it's possible to create a custom element which essentially acts like a standard web element, except that you can use Javascript to control how it behaves, rather than its behaviour being defined by the browser (and if you're lucky the HTML standards). There is a good enough [guide](https://guide.elm-lang.org/interop/custom_elements.html) in the official language docs for how to do this. 

### Use elm-markdown

You can also use the [elm-explorations/markdown](https://package.elm-lang.org/packages/elm-explorations/markdown/latest/) package. Instead of providing the string in Markdown, you just provide it your HTML, but run it with the option to `sanitize` turned off. 

```elm
import Html exposing (Html)
import Markdown

displayHtml : String -> Html msg
displayHtml html =
    let
        defaultOptions =
            Markdown.defaultOptions
        options =
            { defaultOptions | sanitize = False }
    in
    Markdown.toHtml options [] html
```


This basically works because `elm-explorations/markdown` uses kernel code to get around what would otherwise be difficult in Elm code, so this is not something you can replicate in your own package.


### Use iframe + srcdoc

You can use an [iframe](https://developer.cdn.mozilla.net/it/docs/Web/HTML/Element/iframe), and just set the `srcdoc` attribute to the HTML that you wish to display:

```elm
import Html exposing (Html)
import Html.Attributes as Attributes


displayHtml : String -> Html msg
displayHtml html =
    Html.iframe
        [ Attributes.srcdoc html ]
        []
```


## Conclusion

So it is definitely possible to just display arbitrary HTML somewhere in your Elm program. You should be careful that you're not introducing a security bug by displaying arbitrary HTML from somewhere that you do not absolutely trust. So I certainly wouldn't use this to allow users to write comments in HTML for example. It can though, for example, be used to embed some HTML code that is given as embedding code for some service, such as youtube video embed code. 
