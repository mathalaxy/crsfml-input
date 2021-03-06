[![GitHub release](https://img.shields.io/github/release/mathalaxy/crsfml-input.svg)](https://github.com/mathalaxy/crsfml-input/releases)

[![Docs](https://img.shields.io/badge/docs-available-brightgreen.svg)](https://mathalaxy.github.io/crsfml-input/)

# CrSFML-Input
**Customizable keyboard/joystick/mouse input bindings for CrSFML apps**

This shard extends [CrSFML](https://github.com/oprypin/crsfml#readme)'s event-driven and continuous input facilities with customizable bindings. User input is organized around "input events" and "input queries". Which actual input is mapped to these entities then depends on a YAML config file (or a section of another if you prefer), making customization as simple as it gets.

## Installation

1. Add the dependency to your `shard.yml`:

   ```yaml
   dependencies:
     crsfml-input:
       github: mathalaxy/crsfml-input
   ```

2. Run `shards install`

## Usage

To use CrSFML-Input you first need to require it from your project:
```crystal
require "crsfml-input"
# CrSFML-Input requires "crsfml" for you
```

All functions are "static" class methods of the `Input` module which serves as a namespace.

The general idea behind CrSFML-Input is to replace CrSFML's simple mechanism to identify keys/joystick buttons etc. through event classes and realtime access functions by queries to an intermediate layer of *input events* and *input queries* identified by strings. Binding these to particular input devices (keys, buttons, or joystick axes) thus becomes separated from all event/status checks, and in fact from the code as a whole.

For instance, instead of the very specific
```crystal
case key_event.code
when SF::Key::Space
  # toggle pause screen ...
end
```
you could write more generally,
```crystal
case Input.event?(some_event)
when "pause"
  # toggle pause screen ...
end
```
and let the user decide which particular key or key combination should be bound to the "pause" event.

Similarly, you could use `Input.query("run")` to determine the state of whichever key/joystick button/... is bound to "run".
Once you have set up your events and queries, CrSFML-Input will handle the connection between them and the actual user input.

### Defining input events and queries

To make CrSFML-Input work, you first need to define some (abstract) input events and default bindings (concrete keys/buttons). Choose descriptive names for them - something like `toggle pause` or `jump` - and combine them into a YAML structure under the section headings `events` and `queries`, respectively:

```yaml
events:
  step:
    key: Up
  rush:
    key: Ctrl+Shift+Up
    # Ctrl is an alias for SFML's `Control` key code
    # Alt, System, LCtrl, RShift etc. work just the same
  show help:
    # multiple key bindings for the same event:
    key:
    - F1
    - Alt+H
queries:
  charging:
    key: LShift+Space
```

Let CrSFML-Input know about it by setting the `Input.default_bindings` property to a `YAML::Any` or `String`:

```crystal
Input.default_bindings = yaml_or_string
```

If you do not want to embed your defaults within the code, call `Input.load_default_bindings` instead to read directly from a YAML file.

### Custom bindings

Once the *default* bindings are set up for all available input events and queries, you can now easily override some of them with (user) *customized* ones using the `Input.bindings` property:

```crystal
# Override just selected input events
# all other default bindings remain in effect
Input.bindings = <<-NEW_YAML
events:
  show help:
    key: Ctrl+H
NEW_YAML

# Load customized bindings directly from a file
Input.load_bindings "bindtest.yml"

# Use a subsection of a bigger YAML file
cfg = YAMl.parse "config.yml"
if node = cfg.dig("custom", "input", "user-overrides")
  Input.bindings = node
end
```

### Using input events
To handle your abstract input events in a CrSFML app, place a call to `Input.event?` within your event loop and check its return value instead of the SFML event itself:

```crystal
window = SF::Window.new(SF::VideoMode.new(400, 300), "Hi")

while window.open?
  while ev = window.poll_event
    # a nested case switch lets you
    # define the subset of CrSFML
    # events Input should handle
    case ev
    when SF::Event::Closed
      window.close; break
    when SF::Event::MouseButtonClicked
      # handle it
    else
      # Let CrSFML-Input process the event.
      # If a mapping is defined for ev,
      # it will return the input event string.
      case Input.event?(ev)
      when "step"
        # ...
      when "rush"
        # ...
      when "show help"
        # ...
      else
        # return value is nil if no mapping is found
      end
    end
  end

  window.clear
  window.display
end
```

## Development

Planned features list:

- Handle the following *input events*:
  - [x] keyboard events
  - [ ] joystick button events
  - [ ] mouse button events
  - [ ] mouse wheel events
- Handle the following *input queries*:
  - [X] keyboard states
  - [ ] mouse button states
  - [ ] joystick button states
  - [ ] joystick axes states
- [ ] Provide an option to define fallback bindings when e.g. a joystick is disconnected

## Contributing

1. Fork it (<https://github.com/mathalaxy/crsfml-input/fork>)
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

## Contributors

- [mathalaxy](https://github.com/mathalaxy) - creator and maintainer
