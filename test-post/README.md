# Post Processing Example

In addition to providing output parameters by writing files in your Docker image's executable, you can provide simple derived parameters using the post-processing feature.

---

You'll notice that despite the fact that [cmd.py](cmd.py) only writes a single file to provide the value for `test-integer`, the [Dockerfile](Dockerfile) supplies three output parameters: `test-integer`, `test-img-bars`, and `test-integer-plus-one`
```Dockerfile
LABEL mccoy.schema_out "{ \
    \"title\": \"test_post\", \
    \"type\": \"object\", \
    \"properties\": { \
        \"test-integer\": {\"type\": \"integer\", \"title\": \"test-integer\", \"_order\": 1}, \
        \"test-img-bars\": {\"type\": \"string\", \"format\": \"imageSrc\", \"title\": \"test-img-bars\", \"_mime-type\": \"image/jpg\", \"_order\": 2 }, \
        \"test-integer-plus-one\": {\"type\": \"integer\", \"title\": \"test-integer\", \"_order\": 3} \
    } \
}"
```

---

The [Dockerfile](Dockerfile) defines `mccoy.vars.*` `mccoy.postprocess.*` __LABELs__ to provide the values for `test-img-bars` and `test-integer-plus-one`.
```Dockerfile
LABEL mccoy.var.url-1 "https://upload.wikimedia.org/wikipedia/commons/thumb/6/66/SMPTE_Color_Bars.svg/329px-SMPTE_Color_Bars.svg.png"
LABEL mccoy.var.url-2 "https://upload.wikimedia.org/wikipedia/commons/thumb/c/c4/PM5544_with_non-PAL_signals.png/320px-PM5544_with_non-PAL_signals.png"

LABEL mccoy.postprocess.test-img-bars "vars['url-1']"
LABEL mccoy.postprocess.test-integer-plus-one "output['test-integer'] && output['test-integer']+1"
```

Each `mccoy.postprocess.*` is a javascript expression. Each expression will be evaluated and its value set to the output parameter of the same name. Post-process expressions have access to three variables. _Note: a post-process step may change an output value originally provided by the executable._
  * `input` A json object containing a mapping of all input parameters names to their respective value.
  * `output`A json object containing a mapping of all output parameters names to their respective value. _Note: a post-process step may reference a value set by a previous post-process step; post-process steps are evaluated in lexographical order._  
  * `vars` A json object containing a mapping of all `mccoy.var.*` __LABEL__ names to their string values.