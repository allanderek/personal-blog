

I've always said that labels are better than hierachy. So the module system should work in the same manner.
Don't have a hierarchy of modules in which each definition must be in exactly one.
Instead allow definitions to be assigned to multiple modules.
If you want to do the equivalent of `import List exposing (singleton)` in your module `M`, you don't have to
you just assign `singleton` to also be a part of the `M` module. Although that potentially means you can refer to it as `M.singleon`,
but, then, why not?
