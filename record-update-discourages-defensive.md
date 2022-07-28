---
title: "Nested records and defensive programming"
tags: elm syntax programming
date: 2021-02-05
---


A fairly common problem for an Elm developer to encounter after around 6 months is that nested record update is a little painful. Suppose you have a form on your model, like this:

```elm
type alias CommentForm =
    { content : String
    , subject : String
    , visibleToAll : Bool
    }
type alias Model =
    { route : Route
    , commentForm : CommentForm
    , ...
    }
```

So now you want to handle the message for updating the comment form. So you might have a handler like this:


```elm
update message model =
    case message of
        Msg.CommentFormContent input ->
            let
                commentForm =
                    model.commentForm
            in
            ( { model | { commentForm | content = input } }
            , Cmd.none
            )
        ...
```

There are a couple of solutions to this, for example perhaps you should [factor out the messages](/posts/2021-02-03-splitting-up-update) which update only the comment form, or another solution is to avoid the nested record in the first place. Just have all of the comment form fields on the actual main model. If you need to restrict a given function's input for some reason (usually to reuse elsewhere) then you can use extensible record update. So in this case you have:


```elm
type alias Model =
    { route : Route
    { commentFormContent : String
    , commentFormSubject : String
    , commentFormVisibleToAll : Bool
    , ...
    }
update message model =
    case message of
        Msg.CommentFormContent input ->
            ( { model | commentFormContent = input }
            , Cmd.none
            )
        ...
```

For some reason I'm not overly keen on the `commentForm` prefix, somehow I like that being more formal, but anyway the thing I wanted to express today was about defensive programming. Now suppose you want to write the update handler for the successful response to posting the the comment. At this point you want to empty the comment form, ready for the next comment. Let's try this in both styles, first with a nested record:


```elm
-- We write this separately and it can be used in the `init` function to
-- to initialise the comment form on the model.
emptyCommentForm : CommentForm
emptyCommentForm =
    { content = ""
    , subject = ""
    , visibleToAll = False
    }

update message model =
    case message of
    ...
        SubmitCommentFormResponse (Ok _) ->
            ( { model | commentForm = emptyCommentForm }
            , Cmd.none
            )
    ...
```

How would you do this with a flat model structure? Well we can use extensible records to write the `emptyCommentForm`: 

```elm
type alias CommentForm a =
    { a
        | commentFormContent : String
        , commentFormSubject : String
        , commentFormVisibleToAll : Bool
    }
type alias Model =
    CommentForm
        { route : Route
        , ...
        }
emptyCommentForm : CommentForm a -> CommentForm a
emptyCommentForm model =
    { model
        | commentFormContent = ""
        , commentFormSubject = ""
        , commentFormisibleToAll = False
        }

update message model =
    ...
    case message of
        SubmitCommentFormResponse (Ok _) ->
            ( model |> emptyCommentForm
            , Cmd.none
            )
    ...
```

Okay so this is quite nice. I like the update function, you can even define a popular helper function to return the model without any commands and use the right pizza operator:


```elm
update message model =
    let
        noCommands m =
            ( m, Cmd.none )
    in
    ...
    case message of
        SubmitCommentFormResponse (Ok _) ->
            model 
                |> emptyCommentForm
                |> noCommands
    ...
```

However, this approach has two significant drawbacks. I've already hinted at the first. Using nested records the `emptyCommentForm` is not a function but just a record value and can therefore be used to initialise the model in your application's `init` function. Using the extensible record style you cannot use this in your `init` function, you **could** call it with your initial model to make sure that it is always `empty` in the same way, but you still need to input some values for the `commentForm`-prefixed fields. 

The second drawback however, is why I think this style hurts defensive programming **a little**. It concerns adding a field to the comment form. Suppose you wish to add a field which is the comment to which you are replying. In the nested record style:

```elm
type alias CommentForm =
    { content : String
    , subject : String
    , visibleToAll : Bool
    , replyingTo : Maybe CommentId
    }
```

If you make this one change, the Elm compiler will complain to you that your `emptyCommentForm` is no longer valid, because it doesn't define the `replyingTo` field. Fix this and your `init` function **and** the handler for `ubmitCommentFormResponse (Ok _)` are both automatically fixed.

However, if you do this in the extensible record style

```elm
type alias CommentForm a =
    { a
        | commentFormContent : String
        , commentFormSubject : String
        , commentFormVisibleToAll : Bool
        , commentFormReplyingTo : Maybe CommentId
    }
type alias Model =
    CommentForm
        { route : Route
        , ...
        }
```

Unfortunately here you will get no help, because the unchanged `emptyCommentForm` function is still a valid `CommentForm a -> CommentForm a` function.

So for this reason I find the nested record style is **sometimes** the more defensive style. Of course each situation varies and sometimes you have no *'emptying'* of a record to do anyway. Still, this is worth bearing in mind when choosing your data-structures. 


