---
title: "Impossible states and stale messages"
tags: programming elm
date: 2021-03-08
---

There are two quite common pieces of advice for Elm programmers:
1. *Make impossible states impossible*
2. *Avoid carrying state state in your messages*

## Impossible states impossible

The first is a call to carefully consider the types, mostly in your model and messages. So do not have a `Maybe User` type used anywhere where you consider it impossible for the value to be `Nothing`. So for example, you might have a message, such as the *liking of a post*, that requires a logged in user, because the request takes in the authorisation token:

```elm
likePost : AuthToken -> Post.Id -> Cmd Msg
listPost authToken postId =
    ...

type Msg
    = ...
    | LikePost Post.Id
      ...

type alias Model =
    { mUser : Maybe User
    , ...
    }

update : Msg -> Model -> (Model, Cmd Msg)
update message model =
    case message of
        ...
        LikePost postId ->
            let
                token =
                    model.mUser
                        |> Maybe.map .token
                        |> Maybe.withDefault "" -- ARGH
                command =
                    likePost token postId
            in
            ...

viewPost : Model -> Post -> Html Msg
viewPost model post =
    ...
    likeButton =
        case model.mUser of
            Nothing ->
                Html.nothing
            Just _ ->
                Html.button
                    [ LikePost post.Id |> Events.onClick ]
                    [ Html.text "Like" ]
    ...
```

The `-- ARGH` line is supposed to be impossible because we should only render the `like` button if the user is logged in. We can improve this by forcing the view function to check whether the user is logged in, and we can do that by carrying the token or the whole user in the `LikePost` message (using the whole user prevents you from still applying the `LikePost` message with the empty token):


```diff
likePost : AuthToken -> Post.Id -> Cmd Msg
listPost authToken postId =
    ...

type Msg
    = ...
-    | LikePost Post.Id
+    | LikePost User Post.Id
      ...

type alias Model =
    { mUser : Maybe User
    , ...
    }

update : Msg -> Model -> (Model, Cmd Msg)
update message model =
    case message of
        ...
-        LikePost postId ->
+        LikePost user postId ->
            let
-                token =
-                    model.mUser
-                        |> Maybe.map .token
-                        |> Maybe.withDefault "" -- ARGH
                command =
-                    likePost token postId
+                    likePost user.token postId
            in
            ...

viewPost : Model -> Post -> Html Msg
viewPost model post =
    ...
    likeButton =
        case model.mUser of
            Nothing ->
                Html.nothing
-            Just _ ->
+            Just user ->
                Html.button
-                    [ LikePost post.Id |> Events.onClick ]
+                    [ LikePost user post.Id |> Events.onClick ]
                    [ Html.text "Like" ]
    ...
```

Now it's really impossible for the `LikePost` message to be instantiated without a valid user. This seems like a solid win.

## Avoid stale messages

I've [written before](/posts/2021-02-16-stale-messages/) about stale messages. The problem is that you're basically duplicating some of the state of the model in the message, so if you're unfortunate and the model is updated and a second message is invoked before the view is re-rendered, this can lead to bugs. I've found that this can happen when a browser's auto-fill causes a bunch of update messages to be sent essentially simultaneously. In the scenario above, suppose you also want to update the number of likes on the post, something like this:

```elm
update : Msg -> Model -> (Model, Cmd Msg)
update message model =
    case message of
        ...
        LikePost user postId ->
            let
                newPosts =
                    case Dict.get postId model.posts of
                        Nothing ->
                            model.posts -- ARGH
                        Just post ->
                            Dict.insert postId { post | likes = post.likes + 1 } model.posts

                command =
                    likePost user.token postId
            in
            ( { model | posts = newPosts }
            , command
            )
```

Again, the `-- ARGH` line should be impossible. We can do the same trick as above, and pass the entire `post` into to the `LikePost` message:


```diff
type Msg
    = ...
-    | LikePost User Post.Id
+    | LikePost User Post
      ...
update : Msg -> Model -> (Model, Cmd Msg)
update message model =
    case message of
        ...
-        LikePost user postId ->
+        LikePost user post ->
            let
                newPosts =
-                    case Dict.get postId model.posts of
-                        Nothing ->
-                            model.posts -- ARGH
-                        Just post ->
-                            Dict.insert postId { post | likes = post.likes + 1 } model.posts
+                    Dict.insert postId { post | likes = post.likes + 1 } model.posts

                command =
                    likePost user.token postId
            in
            ( { model | posts = newPosts }
            , command
            )

viewPost : Model -> Post -> Html Msg
viewPost model post =
    ...
    likeButton =
        case model.mUser of
            Nothing ->
                Html.nothing
            Just user ->
                Html.button
-                    [ LikePost user post.Id |> Events.onClick ]
+                    [ LikePost user post |> Events.onClick ]
                    [ Html.text "Like" ]
    ...
```

Great, now the whole post is given to the `LikePost` message and we needn't worry about the possibility that the post does not exist. But now your message is carrying possibly stale data. What if, for example, you re-download the list of posts on a periodic basis. If this just happens to arrive in the same animation frame as when the like button is clicked you can get into a race condition where the receipt of the download updates the posts, but the particular post that was liked is then overwritten with stale data. 

You could mitigate this by checking in the dictionary of posts and only using the one in the message if the post is not there. However, you need to consider; is it **really** impossible that the post is not there, if you're re-downloading the list of posts periodically, then that post may have been deleted. The question is what would you want to do in that case? I'd argue that rather than re-inserting the stale data with an updated *like* count, what you want to do is accept that the post has been deleted and just allow that *like* to disappear into the ether. That's what happens with the original code (though the original code would still send the *like* to the server, again that **could** be reasonable behaviour, depending on what the server does if you like a deleted post).

## Conclusion

So both of these mantras are good proverbs to live by. They sometimes conflict, and when they do that means you have a decision to make. It's relatively unlikely that you update the *user* very often or quickly enough for the the stale message data to be a problem (though you could only store the 'token' in the message), so it seems here that guarding against the impossible state is a reasonable choice. However, storing the entire post in the message seems like you're more likely to introduce a stale data bug so perhaps accepting the fact that you cannot type-your-way out of an impossible state is the reasonable choice. Also perhaps it's not so impossible a state anyway.
