A datastore option is a type of variable that can be set by the user, allowing various components of Metasploit to be more configurable during use. For example, in msfconsole, you can set the ConsoleLogging option in order to log all the console input/output - something that's kind of handy for documentation purposes during a pentest. When you load a module, there will be a lot more options registered by the mixin(s) or the module. Some common ones include: RHOST and RPORT for a server-side exploit or auxiliary module, SRVHOST for a client-side module, etc. The best way to find out exactly what datastore options you can set is by using these commands:

* ```show options``` - Shows you all the basic options.
* ```show advanced``` - Shows you all the advanced options.
* ```set``` - Shows you everything. Obviously you also use this command to set an option.

### Option sources: ModuleDataStore, active_module, session, and framework

**How users look at datastore options:**

On the user's side, datastore options are seen as global or module-level: Global means all the modules can use that option, which can be set by using the ```setg``` command. Module-level means only that particular module you're using remembers that datastore option, no other components will know about it. You are setting a module-level option if you load a module first, and then use the ```set``` command, like the following:

```
msf > use exploit/windows/smb/ms08_067_netapi 
msf exploit(ms08_067_netapi) > set rhost 10.0.1.3
rhost => 10.0.1.3
```

**How Metasploit developers look at datastore options:**

On the development side, things are a little crazier. Datastore options actually can be found in at least four different sources: the ModuleDataStore object, active_module, session object, or the framework object.

If you're just doing module development, the best source you can trust is the ModuleDataStore object. This object has a specific load order before handing you the option you want: if the option can be found in the module's datastore, it will give you that. If not found, it will give you the one from framework. The following is an example of how to read a datastore option in a module:

```ruby
current_host = datastore['RHOST']
```

If your dev work is outside the module realm, there is a good possibility that you don't even have the ModuleDataStore object. But in some cases, you still might be able to read from the [active_module accessor](https://github.com/rapid7/metasploit-framework/blob/master/lib/msf/ui/console/driver.rb#L607) from the driver. Or if you have access to [ModuleCommandDispatcher](https://github.com/rapid7/metasploit-framework/blob/master/lib/msf/ui/console/module_command_dispatcher.rb#L28), there is a ```mod``` method too that gives you the same thing, and sometimes mixins pass this around in a ```run_simple``` method while dispatching a module. One example you can look at is the [Msf::Ui::Console::CommandDispatcher::Auxiliary](https://github.com/rapid7/metasploit-framework/blob/master/lib/msf/ui/console/command_dispatcher/auxiliary.rb) class.

In some cases such as running a script in post exploitation, you might not have ModuleDataStore or even active_module, but you should still have a session object. There should be an ```exploit_datastore``` that gives you all the datastore options:

```ruby
session.exploit_datastore
```

If you don't have access to the module, or to a session object, the last source is obviously the framework object, and there is ALWAYS a framework object. However, like we said earlier, if the user sets a model-level option, no other components will see it, this includes the framework object:

```ruby
framework.datastore
```

So now you know there are multiple sources of datastore options. And hopefully at this point you are well aware that not all sources necessarily share the same thing. If you have to try everything, as a general rule, this should be your load order:

1. Try from the ModuleDataStore
2. Try from active_module
3. Try from session
4. Try from framework

### Types of options

All the datastore option types are defined the [option_container.rb](https://github.com/rapid7/metasploit-framework/blob/master/lib/msf/core/option_container.rb) file. You should always pick the most appropriate one because each has its own input validator. The option types are: 

* **OptString** - Typically for a string option. If the input begins with "file://", OptString will also automatically assume this is a file, and read from it. However, there is no file path validation when this happens, so if you want to load a file, you should use the OptPath instead, and then read the file yourself.

* **OptRaw** - It actually functions exactly the same as OptString.

* **OptBool** - Boolean option. It will validate if the input a variant of either true or false. For example: y, yes, n, no, 0, 1, etc.

* **OptEnum** - Basically this will limit to the input to specific choices. For example, if you want the input to be either "apple", or "orange", and nothing else, then OptEnum is the one for you.

* **OptPort** - For an input that's meant to be used as a port number. This number should be between 0 - 65535.

* **OptAddress** - An input that is an IPv4 address.

* OptAddressRange - An input that is a range of IPv4 addresses, for example: 10.0.1.1-10.0.1.20, or 10.0.1.1/24. You can also supply a file path instead of a range, and it will automatically treat that file as a list of IPs. Or, if you do the rand:3 syntax, with 3 meaning 3 times, it will generate 3 random IPs for you.

* **OptPath** - If your datastore option is asking for a local file path, then use this.

* **OptInt** - This can be either a hex value, or decimal.

* **OptRegexp** - Datastore option is a regular expression.

### The register_options method

The ```register_options``` method can register multiple basic datastore options. Basic datastore options are the ones that either required to be configured, such as the RHOST option in a server-side exploit. Or it's very commonly used, such as various username/password options found in a login module.

### The register_advanced_options method

The ```register_advanced_options``` method can register multiple advanced datastore options. Advanced datastore options are the ones that never require the user to configure before using the module. For example, the Proxies option is almost always considered as "advanced". But of course, it can also mean that's something that most user will find difficult to configure.

### The deregister_options method

### Modifying datastore options at run-time