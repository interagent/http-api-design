##### Actions

Prefer endpoint configurations that don’t require special actions. In cases
where actions are needed, clearly dileanate them with the `actions` prefix:

```
/resources/:resource/actions/:action
```

e.g. to stop a particular run:

```
/runs/{run_id}/actions/stop
```

Actions on collections should also be minimized. Where needed, they should use
a top-level actions dilenation to avoid namespace conflicts and clearly show
the scope of action:

```
/actions/:action/resources
```

e.g. to restart all servers:

```
/actions/restart/servers
```
