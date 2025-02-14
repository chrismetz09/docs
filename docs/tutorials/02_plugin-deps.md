# Plugin Dependencies

---

Link to code: [Adding dependencies to your plugin][code-link]

Your control plane agent will typically consist of one or more plugins that
contain the application logic.  In addition there could be other Ligato plugins and components providing various services to your application plugins. Examples include a KV data store, message bus adapters, loggers and health monitors.

This tutorial shows how to add dependencies to your plugins.

Requirements:

* Complete the ['Hello World Agent'](01_hello-world.md) tutorial

The Ligato infrastructure uses the **dependency injection** design pattern to
manage dependencies. In other words, dependencies on other plugins are injected
into your plugin when it is initialized. You should use dependency injection to 
manage all dependencies in your plugin. 

!!! Note
    You need dependency injection to be able to create mocks in your unit tests. This is especially true for components that interact with the external world, such as KV data store or message bus adapters. Without good mocks (i.e. without dependency injection), it is almost impossible to achieve production-level unit test coverage.

One of the most commonly used dependencies in your plugins will likely be 
`PluginDeps` defined in the `github.com/ligato/cn-infra/infra` package. It is
a struct that aggregates three plugin essentials: 

- plugin name
- logging 
- plugin configuration. 

It is defined as:
```go
type PluginDeps struct {
	PluginName
	Log logging.PluginLogger
	Cfg config.PluginConfig
}
```

You embed it into your plugin as follows:

```go
type HelloWorld struct {
	infra.PluginDeps
}
```

`PluginName`, which is embedded in the `PluginDeps` struct, provides the `String()`
method for obtaining the name of the plugin. The plugin name is set using the 
`SetName(name string)` method defined for `PluginName`:

```go
p.SetName("helloworld")
```

The two other components in `PluginDeps` are `Log` and `Cfg`. `Log` is the plugin's logger used to log message at different log levels. `Cfg` is used to load configuration data from a configuration file in YAML format. 

`PluginDeps` has the `Setup()` method, which initializes `Log` and `Cfg` with the name from `PluginName`. It is typically called in the plugin's constructor:

```go
func NewHelloWorld() *HelloWorld {
	p := new(HelloWorld)
	p.SetName("helloworld")
	p.Setup()
	return p
}
```

After initializing `Log` and `Cfg`, they are ready for action. 

Let's log a few messages:

```go
func (p *HelloWorld) Init() error {
	p.Log.Info("System ready.")
	p.Log.Warn("Problems found!")
	p.Log.Error("Errors encountered!")
}
```

For more details on the Log API see [infra/logging/log_api.go](https://github.com/ligato/cn-infra/blob/master/logging/log_api.go).

Now, let's load configuration from a file. By default, the name of the config file will be derived from the plugin name with extension `.conf`. In our example, the configuration file name will be `helloworld.conf`.

```go
type Config struct {
	MyValue int `json:"my-value"`
}

func (p *HelloWorld) Init() error {
	cfg := new(Config)
	found, err := p.Cfg.LoadValue(cfg)
	// ...
}
```

If the config file is not found, the `LoadValue` will return false. If the configuration cannot be parsed, the function will return an error.

__Run the Plugin Dependency code__

```
go run main.go
```
Example output
```
INFO[0000] Starting agent version: v0.0.0-dev            BuildDate= CommitHash= loc="agent/agent.go(134)" logger=agent
INFO[0000] Greetings World!                              loc="02_plugin-deps/main.go(77)" logger=helloworld
INFO[0000] Agent started with 1 plugins (took 1ms)       loc="agent/agent.go(179)" logger=agent
```

Example output upon `ctrl-c user interrupt`
```
^CINFO[0013] Signal interrupt received, stopping.          loc="agent/agent.go(196)" logger=agent
INFO[0013] Stopping agent                                loc="agent/agent.go(269)" logger=agent
INFO[0013] Goodbye World!                                loc="02_plugin-deps/main.go(83)" logger=helloworld
INFO[0013] Agent stopped                                 loc="agent/agent.go(291)" logger=agent
```


The complete working example can be found at [examples/tutorials/02_plugin-deps](https://github.com/ligato/cn-infra/blob/master/examples/tutorials/02_plugin-deps).

[code-link]: https://github.com/ligato/cn-infra/tree/master/examples/tutorials/02_plugin-deps