task :generate, [:framework_name, :organization_name, :bundle_identifier_prefix] do |t, args|

  name = args.framework_name
  organization_name = args.organization_name
  bundle_identifier_prefix = args.bundle_identifier_prefix

  task_format = '"generate[:framework_name, :organization_name, :bundle_identifier_prefix]"'
  if !name
    abort("Missing framework_name ()#{task_format})")
    return
  end

  if !organization_name
    abort("Missing organization_name (#{task_format})")
  end

  if !bundle_identifier_prefix
    abort("Missing bundle_identifier_prefix (e.g. com.company) #{task_format}")
  end

  if Dir.exists?(name)
    abort("Directory '#{name}' already exists")
  end
  require 'fileutils'

  FileUtils.mkdir(name)

  Dir.chdir(Dir.pwd + '/' + name) do
    createProject(name, organization_name, bundle_identifier_prefix)
  end
end

private

def createProject(name, organization_name, bundle_identifier_prefix)

  require 'xcodeproj'

  # Create project
  proj = Xcodeproj::Project.new(name + '.xcodeproj')
  proj.root_object.attributes['ORGANIZATIONNAME'] = organization_name

  # Create targets
  framework_target = proj.new_target(:framework, name, :ios, '9.0')
  test_target = createTestTarget(proj, framework_target)

  # Create directories
  FileUtils.mkdir(framework_target.name)
  FileUtils.mkdir(test_target.name)

  # Setup dependencies
  templates_directory = '../Templates/'
  FileUtils.cp(templates_directory + 'Cartfile.private', 'Cartfile.private')
  sh('carthage bootstrap')

  # Create groups
  group_framework = proj.new_group(framework_target.name)
  group_framework.set_path(framework_target.name)
  group_framework_supporting_files = group_framework.new_group('Supporting Files')
  group_tests = proj.new_group(test_target.name)
  group_tests.set_path(test_target.name)
  group_tests_supporting_files = group_tests.new_group('Supporting Files')
  group_config = proj.new_group('Configuration')
  group_base = group_config.new_group('Base')
  group_base_configs = group_base.new_group('Configurations')
  group_targets = group_base.new_group('Targets')
  group_ios = group_config.new_group('iOS')

  # Add umbrella header
  template_variables = {
    :FRAMEWORK_NAME => framework_target.name,
    :DEVELOPER_NAME => `git config --get user.name`.delete!("\n"),
    :ORGANIZATION => organization_name,
    :DATE => `date +"%m/%d/%y"`.delete!("\n"),
    :YEAR => `date +"%Y"`.delete!("\n")
  }
  umbrella_header_filename = framework_target.name + '.h'
  umbrealla_header_path = framework_target.name + '/' + umbrella_header_filename
  copyFileWithTemplate(templates_directory + 'Framework.h', umbrealla_header_path, template_variables)
  umbrella_header_file = group_framework.new_file(umbrella_header_filename)
  framework_target.add_file_references([umbrella_header_file])

  # make umbrella header public
  umbrella_header_file.build_files.each do |file|
    file.settings = { 'ATTRIBUTES' => ['Public'] }
  end

  # Add Info.plist files
  info_plist_filename = 'Info.plist'
  info_plist_path = framework_target.name + '/' + info_plist_filename
  info_plist_vars = {:BUNDLE_IDENTIFIER_PREFIX => bundle_identifier_prefix}
  copyFileWithTemplate(templates_directory + info_plist_filename, info_plist_path, info_plist_vars)
  group_framework_supporting_files.new_file(info_plist_filename)
  framework_target.build_settings('Debug')['INFOPLIST_FILE'] = info_plist_path
  framework_target.build_settings('Release')['INFOPLIST_FILE'] = info_plist_path

  test_info_plist_path = test_target.name + '/' + info_plist_filename
  copyFileWithTemplate(templates_directory + 'InfoTests.plist', test_info_plist_path, info_plist_vars)
  group_tests_supporting_files.new_file(info_plist_filename)
  test_target.build_settings('Debug')['INFOPLIST_FILE'] = test_info_plist_path
  test_target.build_settings('Release')['INFOPLIST_FILE'] = test_info_plist_path

  # Add https://github.com/jspahrsummers/xcconfigs config files
  framework_config_path = framework_target.name + '/' + framework_target.name + '.xcconfig'
  copyFileWithTemplate(templates_directory + 'Framework.xcconfig', framework_config_path, template_variables)

  tests_config_path = test_target.name + '/' + test_target.name + '.xcconfig'
  test_template_variables = template_variables
  test_template_variables[:TESTS_TARGET_NAME] = test_target.name
  copyFileWithTemplate(templates_directory + 'FrameworkTests.xcconfig', tests_config_path, test_template_variables)

  framework_config = group_config.new_file(framework_config_path)
  tests_config = group_config.new_file(tests_config_path)

  configs_path = 'Carthage/Checkouts/xcconfigs/'
  group_base.new_file(configs_path + 'Base/Common.xcconfig')
  debug_config = group_base_configs.new_file(configs_path + 'Base/Configurations/Debug.xcconfig')
  group_base_configs.new_file(configs_path + 'Base/Configurations/Profile.xcconfig')
  release_config = group_base_configs.new_file(configs_path + 'Base/Configurations/Release.xcconfig')
  group_base_configs.new_file(configs_path + 'Base/Configurations/Test.xcconfig')
  group_targets.new_file(configs_path + 'Base/Targets/Application.xcconfig')
  group_targets.new_file(configs_path + 'Base/Targets/Framework.xcconfig')
  group_targets.new_file(configs_path + 'Base/Targets/StaticLibrary.xcconfig')
  group_ios.new_file(configs_path + 'iOS/iOS-Application.xcconfig')
  group_ios.new_file(configs_path + 'iOS/iOS-Base.xcconfig')
  group_ios.new_file(configs_path + 'iOS/iOS-Framework.xcconfig')
  group_ios.new_file(configs_path + 'iOS/iOS-StaticLibrary.xcconfig')

  # Set build configurations
  proj.build_configuration_list['Debug'].base_configuration_reference = debug_config
  proj.build_configuration_list['Release'].base_configuration_reference = release_config

  framework_target.build_configuration_list['Debug'].base_configuration_reference =  framework_config
  framework_target.build_configuration_list['Release'].base_configuration_reference =  framework_config

  test_target.build_configuration_list['Debug'].base_configuration_reference =  tests_config
  test_target.build_configuration_list['Release'].base_configuration_reference =  tests_config

  proj.save
end

def copyFileWithTemplate(source_path, desintation_path, template_variables)
  source_file_contents = File.read(source_path)
  new_contents = source_file_contents % template_variables
  destination_file = File.open(desintation_path, 'w')
  destination_file.write(new_contents)
end

# See https://groups.google.com/forum/#!topic/cocoapods/feDhJjLWu48
def createTestTarget(proj, framework_target)
  test_target = proj.new(Xcodeproj::Project::PBXNativeTarget)
  proj.targets << test_target
  test_target.name = framework_target.name + 'Tests'
  test_target.product_name = test_target.name
  test_target.product_type = 'com.apple.product-type.bundle.unit-test'
  test_target.build_configuration_list = Xcodeproj::Project::ProjectHelper.configuration_list(proj, :ios, '9.0')

	product_ref = proj.products_group.new_reference(test_target.name + '.xctest', :built_products)
	product_ref.include_in_index = '0'
	product_ref.set_explicit_file_type
	test_target.product_reference = product_ref

	test_target.build_phases << proj.new(Xcodeproj::Project::PBXSourcesBuildPhase)
	test_target.build_phases << proj.new(Xcodeproj::Project::PBXFrameworksBuildPhase)
	test_target.build_phases << proj.new(Xcodeproj::Project::PBXResourcesBuildPhase)

	test_target.add_dependency(framework_target)

  return test_target
end
