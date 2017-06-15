Chassis
=======

Chassis generates iOS framework projects that leverage [jspahrsummers/xcconfigs](https://github.com/jspahrsummers/xcconfigs) for build configurations. Projects generated with Chassis also contain a testing target.

Dependencies
------------

### [Xcodeproj](https://github.com/CocoaPods/Xcodeproj)
`$ [sudo] gem install xcodeproj`

### [Carthage](https://github.com/Carthage/Carthage)
`$ brew install carthage`

Usage
-----
When given a _framework_name_, _organization_name_, _bundle_id_prefix_, and optionally an _output_dir_ (defaults to _framework_name_) Chassis will populate the _output_dir_ with the iOS framework project.

### Syntax
`:generate, [:framework_name, :organization_name, :bundle_id_prefix, :output_dir]`

### Example
`rake "generate[MyFramework, MyOrganization, com.myorganization, /Some/Folder]"`
