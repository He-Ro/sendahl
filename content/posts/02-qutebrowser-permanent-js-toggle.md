Title: My JavaScript toggle in qutebrowser
Date: 2018-08-10 11:00
Modified: 2018-08-10 11:00
Category: configs
Tags: qutebrowser, js
Slug: qutebrowser-javascript-config
Authors: Hendrik Rosendahl
Summary: My configuration of qutebrowser to toggle JavaScript similar to NoScript

This post is about [qutebrowser](https://www.qutebrowser.org/) the browser I am currently mainly using.
It supports being controlled mainly with the keyboard and the keybindings are inspired by vim, which is a nice feature.

To configure qutebrowser you can either do it with commands such as `:set config <value>` or `:bind 'xx' 'awesome command'`.
Those configurations are stored in a `autoconfig.yml` file which gets loaded by qutebrowser.
For more advanced configurations such as reading colors from Xresources one can also use a `config.py` file, which is essentially a Python file that modifies the special variables `c` and `config`.
So instead of doing `:set content.javascript.enabled False` in the browser, you would add `c.content.javascript.enabled = False` in the Python config.
Now if you use a `config.py` file the `autoconfig.yml` file is no longer read automatically.
You can still use it by explicitly loading it in your `config.py` file, but I like to not use the `autoconfig.yml` file.
This way I can experiment in the browser by setting some configuration options or adding key bindings and they disappear after I close qutebrowser.

*(Note: For `:set` there exists the `--temp` flag, which acchieves a similar effect, but for bind it is not presently available and also I often forgot to add the `--temp` flag)*

So I added the following to my `config.py` file:
```python
c.javascript.enabled = False

# Load allowed javascript hosts at startup
ALLOWED_HOSTS_FILE = config.configdir / 'javascript_allowed_hosts.txt'                                                                        
with open(ALLOWED_HOSTS_FILE) as allowed_hosts:
    for pattern in allowed_hosts:
        pattern = pattern.strip()
        config.set('content.javascript.enabled', True, '{}'.format(pattern))                                                                  

# Add required bindings
JS_TOGGLE = config.configdir / 'javascript_toggle.sh'
SPAWN_CALL = "spawn --userscript {} {{}}".format(JS_TOGGLE)
config.bind("tSh", SPAWN_CALL.format("*://{url:host}/*"))
config.bind("tSH", SPAWN_CALL.format("*://*.{url:host}/*"))
config.bind("tSu", SPAWN_CALL.format("{url}"))
```
And I created a new file in the configuration directory named `javascript_toggle.sh` with the following content:
```bash
#! /bin/bash

CURRENT_DIR=$(dirname $0)
ALLOWED_FILE=${CURRENT_DIR}/javascript_allowed_hosts.txt

if [ ! $# == 1 ]; then
        exit 1
fi

PATTERN=$1
if grep -qF "${PATTERN}" ${ALLOWED_FILE}; then
        # Remove the Host from the allowed file
        sed -i "\}^${PATTERN}$}d" ${ALLOWED_FILE}
        echo "set -p -t --pattern ${PATTERN} content.javascript.enabled False" > ${QUTE_FIFO}
else
        # Add the Host to the allowed file
        echo ${PATTERN} >> ${ALLOWED_FILE}
        echo "set -p -t --pattern ${PATTERN} content.javascript.enabled True" > ${QUTE_FIFO}
fi
echo reload > ${QUTE_FIFO}
```

*(Note: Do not forget to make the script executable)*

The script keeps track of all hosts that are allowed to use JavaScript and also toggles the same setting in the running qutebrowser session by piping the command into the special file `QUTE_FIFO`.

The snippet for the configuration file loads the accumulated patterns on startup and allows JavaScript only for those patterns.
Furthermore it changes the bindings `tSh`, `tSH` and `tSu` to use the new script.
Those new bindings are derived from the default bindings, which essentially do the same thing, but the default bindings only add the patterns to the `autoconfig.yml` file and with this change the patterns are stored in the file `javascript_allowed_hosts.txt`.
