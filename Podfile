require File.join(File.dirname(`node --print "require.resolve('expo/package.json')"`), "scripts/autolinking")
require File.join(File.dirname(`node --print "require.resolve('react-native/package.json')"`), "scripts/react_native_pods")

require 'json'
podfile_properties = JSON.parse(File.read(File.join(__dir__, 'Podfile.properties.json'))) rescue {}

ENV['RCT_NEW_ARCH_ENABLED'] = podfile_properties['newArchEnabled'] == 'true' ? '1' : '0'
ENV['EX_DEV_CLIENT_NETWORK_INSPECTOR'] = podfile_properties['EX_DEV_CLIENT_NETWORK_INSPECTOR']

platform :ios, podfile_properties['ios.deploymentTarget'] || '15.5'
install! 'cocoapods',
  :deterministic_uuids => false

prepare_react_native_project!

target 'SplitDontArgue' do
  use_expo_modules!
  config = use_native_modules!

  use_frameworks! :linkage => :static

  # Flags to help RNFB work with static frameworks
  $RNFirebaseAsStaticFramework = true

  # React Native Firebase pods
  pod 'RNFBApp', :path => '../node_modules/@react-native-firebase/app'
  pod 'RNFBAuth', :path => '../node_modules/@react-native-firebase/auth'
  pod 'RNFBFirestore', :path => '../node_modules/@react-native-firebase/firestore'
  pod 'RNFBStorage', :path => '../node_modules/@react-native-firebase/storage'
  pod 'RNFBFunctions', :path => '../node_modules/@react-native-firebase/functions'

  use_react_native!(
    :path => config[:reactNativePath],
    :hermes_enabled => true,
    :app_path => "#{Pod::Config.instance.installation_root}/..",
    :privacy_file_aggregation_enabled => true,
  )

  post_install do |installer|
    react_native_post_install(installer, config[:reactNativePath], :mac_catalyst_enabled => false)
    
    # Fix for gRPC compilation in frameworks
    installer.pods_project.targets.each do |target|
      target.build_configurations.each do |config|
        # Allow non-modular includes for everything
        config.build_settings['CLANG_ALLOW_NON_MODULAR_INCLUDES_IN_FRAMEWORK_MODULES'] = 'YES'
        
        # Force Deployment Target to match top-level (fix for gRPC/Libtool errors)
        config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '15.5'
        
        # Disable Bitcode (deprecated but safe to force off)
        config.build_settings['ENABLE_BITCODE'] = 'NO'
        
        # Disable Module Verification (fixes leveldb/socketrocket failures)
        config.build_settings['ENABLE_MODULE_VERIFIER'] = 'NO'
      end
    end
  end
end
