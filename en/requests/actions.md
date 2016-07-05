##### Actions

Prefer endpoint layouts that don’t need any special actions for
individual resources. In cases where special actions are needed, place
them under a standard `actions` prefix, to clearly delineate them:

```
/resources/:resource/actions/:action
```

e.g.

```
/runs/{run_id}/actions/stop
```

Occasionally operations on a whole collection may also be required, here
`actions` prefixes the resource to clarify the target:

```
/actions/:action/resources
```

e.g.

```
/actions/stop/runs
```
