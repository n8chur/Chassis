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
When given a _framework_name_, _organization_name_, and _bundle_id_prefix_, Chassis will generate a folder with the same name as the _framework_name_ and populate it with the iOS framework project.

### Syntax
`:generate, [:framework_name, :organization_name, :bundle_id_prefix]`

### Example
`rake "generate[MyFramework, MyOrganization, com.myorganization]"`
