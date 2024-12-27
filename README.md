ECSalt - Entity Component System for Erlang
===========================================

ECSalt (pronounced Exalt) is an Entity-Component-System-like library for Erlang
applications.

## Using ECSalt
For a contrived example, we'll go through adding a system that makes a player
take damage if they are on fire.

First start ECSalt:
```erlang
1> World = ecsalt_gs:start().
{world,[],#Ref<0.1960388004.2533228547.251798>,
       #Ref<0.1960388004.2533228547.251799>}
```

Suppose we have a fireplace and it has the burning state. We represent the
Fireplace with the ID 1:
```erlang
2> Fireplace = 1.
3> ecsalt_gs:add_component(burning, true, Fireplace, World).
```

Now let's imagine an alley cat snuggles a bit too close to the fireplace and
starts smoldering:
```erlang
4> Cat = 2.
5> ecsalt_gs:add_component(burning, true, Cat, World).
ok
```

You can give the cat a few more properties with the plural `add_components/3`
function:
```erlang
6> ecsalt_gs:add_components([{hp, 100}, {color, orange}, {brain_cells, 1}], Cat, World).
```

Now suppose want to check for all entities that are on fire and have an HP
(health points) component. We can use the `match_components/2` function that
will *only* return the functions that match all required components. For
example, our toasty orange cat matches here, but the fireplace does not because
it doesn't have HP.
```erlang
7> ecsalt_gs:match_components([hp, burning], World).
[{2, [{hp, 100}, {color, orange}, {brain_cells, 1}, {burning, true}]}]
```

We can also define systems that will act on collections of components. Systems
must be one of: `fun` with arity of 2, a `mfa()` tuple of the form {Module,
Fun, Arity} where the function's arity can be 1 or 2. Suppose we have a system
that checks if the cat is on fire and updates their HP accordingly:
```erlang
8> G = fun({ID, Components}) ->
      HP = ecsalt_gs:get(hp, Components),
      if HP < 100 -> io:format("Kitty on fire!!");
         true -> io:format("Kitty is dead!")
      end,
      ecsalt_gs:add_component(hp, HP - 10, ID, World)
    end.
#Fun<erl_eval.41.39164016>
10> System = fun(_Data, World) ->
      Matches = ecsalt_gs:match_components([hp, burning], World),
      lists:foreach(G, Matches)
    end.
#Fun<erl_eval.41.39164016>
11> ecsalt_gs:add_system(System, World).
{ok,{world,[{100,#Fun<erl_eval.41.39164016>}],
           #Ref<0.1960388004.2533228547.253612>,
           #Ref<0.1960388004.2533228547.253613>}}
```

Now you can 'proc' (a term borrowed from multi-user dungeons) this system
periodically:
```erlang
12> ecsalt_gs:proc(World1).
[{#Fun<erl_eval.41.39164016>,ok}]
```
