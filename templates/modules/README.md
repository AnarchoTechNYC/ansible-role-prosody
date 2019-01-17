# Prosody module configuration templates

The `*.cfg.lua.j2` files in this folder are template fragments (`include`'d files) loaded by a parent template. Each template corresponds to the implementation of the various configuration options made available by the Prosody plugin ("module") named by the template file. For example, [the `mod_register.cfg.lua.j2` file](mod_register.cfg.lua.j2) implements the various options made available by [Prosody's `register` module](https://prosody.im/doc/modules/mod_register).
