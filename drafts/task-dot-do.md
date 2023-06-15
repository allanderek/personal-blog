

flip : (a -> b) -> b -> a

andThen : (a -> Task x b) -> Task x a -> Task x b

Task.do : Task x a -> (a -> Task x b) -> Task x b

Task.do (getUserFromLocalStorage "Robin") \user ->
    Task.do (sendUserToHttpServer user) \response ->
         Task.do (saveUserToLocalStorage response.user) \ok ->
             Task.succeed ok

getUserFromLocalStorage "Robin"
  |> Task.andThen sendUserToHttpServer
  |> Task.andThen (.user >> saveUserToLocalStorage)

getUserFromLocalStorage "Robin"
  |> Task.andThen sendUserToHttpServer
  |> Task.andThen (\response -> saveUserToLocalStorage response.user)
