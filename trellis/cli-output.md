Trellis expands your control over Ansible's CLI output in terms of format, verbosity, and color. The options below may be applied via `ansible.cfg` or via environment variables. When an environment variable is set, it will take priority.

## Output format
### Custom output
* Example: `stdout_callback = output`
* Under `[defaults]` in `ansible.cfg`
* To disable: Comment out or remove `stdout_callback = output`
* Default: Enabled
* Reference: [`stdout_callback`](http://docs.ansible.com/ansible/intro_configuration.html#stdout-callback) in Ansible docs

When custom output is enabled, Trellis will attempt to pretty print Ansible's CLI output. This mostly consists of extracting values such as`msg` and `stdout`, interpretting `\n` line breaks, wrapping text to the `wrap_width` described below, and reducing output verbosity according to the options described below.

### `wrap_width`
* Example: `wrap_width = 80`
* Under `[trellis_output]` in `ansible.cfg`
* Default if omitted: `80`
* Optional environment variable: `TRELLIS_WRAP_WIDTH`

`wrap_width` controls the textwrap width applied to the CLI output (number of columns).

## Output verbosity
Trellis adds a few custom options to minimize Ansible output. The optional environment variables have priority over parameters defined in `ansible.cfg`.

If your command includes a verbosity option of `-v` or greater, the play will run as though these parameters are set to their more verbose setting. However, the `display_skipped_hosts` parameter will be unaffected because it is an Ansible core parameter, not controlled by Trellis.

### `display_include_tasks`
* Example: `display_include_tasks = False`
* Under `[trellis_output]` in `ansible.cfg`
* Default if omitted: `False`
* Optional environment variable: `TRELLIS_DISPLAY_INCLUDE_TASKS`

```
# output when display_include_tasks = True
TASK [deploy : include] ********************************************************
included: /path/example.com/ansible/roles/deploy/tasks/initialize.yml for HOST

# all include task output is omitted when display_include_tasks = False

```

### `display_skipped_items`
* Example: `display_skipped_items = False`
* Under `[trellis_output]` in `ansible.cfg`
* Default if omitted: `False`
* Optional environment variable: `TRELLIS_DISPLAY_SKIPPED_ITEMS`

```
# output when display_skipped_items = True
TASK [Demonstrate display_skipped_items -- in ansible.cfg trellis_output] ******
changed: [localhost] => (item={u'msg': u'Foo message', u'name': u'Foo'})
skipping: [localhost] => (item={u'msg': u'Displays only when display_skipped_items = True', u'name': u'Bar'})

# output when display_skipped_items = False
TASK [Demonstrate display_skipped_items -- in ansible.cfg trellis_output] ******
changed: [localhost] => (item={u'msg': u'Foo message', u'name': u'Foo'})

```

### `display_skipped_hosts`
* Example: `display_skipped_hosts = False`
* Under `[defaults]` in `ansible.cfg`
* Default if omitted: `True`
* Optional environment variable: `DISPLAY_SKIPPED_HOSTS`
* Reference: [`display_skipped_hosts`](http://docs.ansible.com/ansible/intro_configuration.html#display-skipped-hosts) in Ansible docs

```
# output when display_skipped_hosts = True
TASK [Demonstrate display_skipped_hosts -- in ansible.cfg defaults] ************
skipping: [localhost]

# output when display_skipped_hosts = False
TASK [Demonstrate display_skipped_hosts -- in ansible.cfg defaults] ************

```

### `truncate_items`
* Example: `truncate_items = True`
* Under `[trellis_output]` in `ansible.cfg`
* Default if omitted: `True`
* Optional environment variable: `TRELLIS_TRUNCATE_ITEMS`

Tasks that loop over items (e.g., `with_items` and `with_dict`) typically print the content of each item. The output can look rather cluttered when the printed items contain a lot of content. The `truncate_items` option will truncate each item's content to `wrap_width` or to just the item key if the item is a dict.

```
# output when truncate_items = False
TASK [Demonstrate truncate_items  -- in ansible.cfg trellis_output] ************
ok: [localhost] => (item={u'msg': u'This item object will only display in all its longline glory if `truncate_items = False` or if the item fails', u'name': u'Foo'})
ok: [localhost] => (item={'value': {u'multisite': {u'enabled': False}, u'cache': {u'enabled': False}, u'repo': u'git@github.com:roots/bedrock.git', u'ssl': {u'enabled': False, u'provider': u'letsencrypt'}, u'local_path': u'../site', u'branch': u'master', u'site_hosts': [u'example.com']}, 'key': u'example.com'})

# output when truncate_items = True
TASK [Demonstrate truncate_items  -- in ansible.cfg trellis_output] ************
ok: [localhost] => (item={u'msg': u'This item object will only display in al...)
ok: [localhost] => (item=example.com)
```

Exceptions
* On item failed: Item output will display in full for failed items, for the sake of easier debugging.
* On item skipped: Item output will be completely absent if the item skips and `display_skipped_items = True` or if a host skips and `display_skipped_hosts = True`.

## Output color
Trellis applies Ansible's built-in colors to the output formatting described above. Your terminal emulator likely has options to further adjust these colors (e.g., to adjust contrast, etc.). The optional environment variables have priority over parameters defined in `ansible.cfg`. You may set any color to `None` and the corresponding text will inherit the color of its parent element.

### Color examples
To help you see all available colors (Ansible's [`codeCodes`](https://github.com/ansible/ansible/blob/devel/lib/ansible/utils/color.py)) and customize your color settings, copy the code below to a file named `output_colors.yml` in the root of your Trellis project and run `ansible-playbook output_colors.yml`. If desired, adjust colors in `ansible.cfg` and run the playbook again to see how the change looks.

```
- name: Colors
	gather_facts: false
	hosts: localhost
	tasks:
		- name: Show available Ansible colors and current color settings
			debug:
				msg: |
					AVAILABLE COLORS
					----------------
					{{ ansible_colors }}

					CURRENT COLOR SETTINGS
					----------------------
					{{ color_settings }}
		- name: Example fail message
			fail:
				msg: This demonstrates a failed task with system info, hr, and error message.
```

### `color_code`
* Example: `color_code = normal`
* Under `[trellis_output]` in `ansible.cfg`
* Default if omitted: `normal`
* Optional environment variable: `TRELLIS_COLOR_CODE`
* To disable: `color_code = None` (when `None`, backticks will display around code text)

### `color_code_block`
* Example: `color_code_block = yellow`
* Under `[trellis_output]` in `ansible.cfg`
* Default if omitted: Ansible environment variable `COLOR_CHANGED` (usually `yellow`)
* Optional environment variable: `TRELLIS_COLOR_CODE_BLOCK`
* To disable: `color_code_block = None`

### `color_default`
* Example: `color_default = cyan`
* Under `[trellis_output]` in `ansible.cfg`
* Default if omitted: Ansible environment variable `COLOR_SKIP` (usually `cyan`)
* Optional environment variable: `TRELLIS_COLOR_DEFAULT`
* To disable: `color_default = None`

### `color_error`
* Example: `color_error = red`
* Under `[trellis_output]` in `ansible.cfg`
* Default if omitted: Ansible environment variable `COLOR_ERROR` (usually `red`)
* Optional environment variable: `TRELLIS_COLOR_ERROR`
* To disable: `color_error = None`

### `color_hr`
* Example: `color_hr = bright gray`
* Under `[trellis_output]` in `ansible.cfg`
* Default if omitted: `bright gray`
* Optional environment variable: `TRELLIS_COLOR_HR`
* To disable: `color_hr = None`

### `color_ok`
* Example: `color_ok = green`
* Under `[trellis_output]` in `ansible.cfg`
* Default if omitted: Ansible environment variable `COLOR_OK` (usually `green`)
* Optional environment variable: `TRELLIS_COLOR_OK`
* To disable: `color_ok = None`

### `color_system_info`
* Example: `color_system_info = bright gray`
* Under `[trellis_output]` in `ansible.cfg`
* Default if omitted: `bright gray`
* Optional environment variable: `TRELLIS_COLOR_SYSTEM_INFO`
* To disable: `color_system_info = None`

### `color_warn`
* Example: `color_warn = bright purple`
* Under `[trellis_output]` in `ansible.cfg`
* Default if omitted: Ansible environment variable `COLOR_WARN` (usually `bright purple`)
* Optional environment variable: `TRELLIS_COLOR_WARN`
* To disable: `color_warn = None`

## Templating
Trellis will apply the colors settings from the "Output color" section above to the following elements:
* Color strings
	* Color strings are strings wrapped in `<error>text</error>`, `<warn>text</warn>`, `<ok>text</ok>`, `<bold>text</bold>`, or `<default>text</default>`.
	* You may nest color strings inside any element, including inside other color strings.
	* `<bold>` just prepends 'bright' onto the color name, or changes 'dark gray' to 'bright gray'. Thus `<bold>` will have no effect when the parent element's color name starts with 'bright' or is 'black', 'normal', or 'white'.
* Code strings
	* Code strings are `strings wrapped in backticks`.
	* You may nest code strings inside any element.
* Code blocks
	* A code block is text fenced off before and after by a new line with ```.
	* You may not nest code blocks inside any other elements.
	* No text wrapping is applied to code blocks.

Although elements may nest, they may not overlap. A child element's opening and closing tags must both be inside the parent's opening and closing tags.
```
# Yes (<error> is completely nested)
`Code string with <error>error value</error> completely nested`

# No (<bold> is not completely nested)
Text string <ok>with <bold>overlapped</ok> color strings</bold>
```
