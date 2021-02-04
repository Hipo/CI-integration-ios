fastlane_version "2.72.0"

default_platform(:ios)

platform :ios do

  shared_env_variables = nil

  tryouts_prod_release = nil
  tryouts_preprod_release = nil
  tryouts_staging_release = nil

  before_all do |lane, options|
    shared_env_variables = [
      "FASTLANE_USER",
      "TEAM_ID",
      "KEY_ID",
      "ISSUER_ID",
      "KEY_CONTENT",
      "IN_HOUSE",
      "WORKSPACE",
      "SLACK_WEBHOOK_URL",
      "MATCH_PASSWORD",
      "S3_BUCKET",
      "S3_ACCESS_KEY",
      "S3_SECRET_ACCESS_KEY",
      "S3_REGION",
      "APP_NAME"
    ]
  end

  # PUBLIC LANES

  lane :build_store_app do |options|
    check_store_env_variables
    
    build(
      app_identifier: ENV["APP_ID"],
      scheme: ENV["SCHEME"],
      is_store: true
    )
  end

  lane :build_staging_app do |options|
    check_staging_env_variables

    build(
      app_identifier: ENV["STAGING_APP_ID"],
      scheme: ENV["STAGING_SCHEME"],
      is_store: false
    )
  end

  lane :build_preprod_app do |options|
    check_preprod_env_variables

    build(
      app_identifier: ENV["PREPROD_APP_ID"],
      scheme: ENV["PREPROD_SCHEME"],
      is_store: false
    )
  end

  lane :build_prod_app do |options|
    check_prod_env_variables

    build(
      app_identifier: ENV["APP_ID"],
      scheme: ENV["PROD_SCHEME"],
      is_store: false
    )
  end

  lane :deploy_all_apps_to_tryouts do |options|
    deploy_staging_app_to_tryouts(send_notification: false)
    deploy_preprod_app_to_tryouts(send_notification: false)
    deploy_prod_app_to_tryouts(send_notification: false)

    actions = []

    if tryouts_staging_release != nil
      actions.push(tryouts_staging_release)
    end

    if tryouts_preprod_release != nil
      actions.push(tryouts_preprod_release)
    end

    if tryouts_prod_release != nil
      actions.push(tryouts_prod_release)
    end

    send_tryouts_all_notification_to_slack(actions: actions)
  end

  lane :deploy_staging_app_to_tryouts do |options|    
    send_notification = options[:send_notification] || true
    target = "Staging"
    app_id = ENV["STAGING_APP_ID"]

    if app_id == nil
      UI.important "No app is found to deploy to Tryouts [#{target}]"
      next
    end

    if send_notification
      check_staging_env_variables
    end

    deploy_to_tryouts(
      target: target,
      app_identifier: app_id,
      scheme: ENV["STAGING_SCHEME"],
      ipa_name: ENV["STAGING_IPA_NAME"],
      tryouts_app_id: ENV["STAGING_TRYOUTS_APP_ID"],
      tryouts_api_token: ENV["STAGING_TRYOUTS_API_TOKEN"],
      google_service_info_plist_path: ENV["STAGING_GOOGLE_SERVICE_INFO_PLIST_PATH"]
    )

    tryouts_release = lane_context[SharedValues::TRYOUTS_BUILD_INFORMATION] 

    tryouts_staging_release = {text: app_name_for_target(target: target), url: tryouts_release["download_url"]}

    if send_notification
      send_tryouts_target_notification_to_slack(target: target)
    end
  end

  lane :deploy_preprod_app_to_tryouts do |options|
    send_notification = options[:send_notification] || true
    target = "Preprod"
    app_id = ENV["PREPROD_APP_ID"]

    if app_id == nil
      UI.important "No app is found to deploy to Tryouts [#{target}]"
      next
    end

    if send_notification
      check_preprod_env_variables
    end

    deploy_to_tryouts(
      target: target,
      app_identifier: app_id,
      scheme: ENV["PREPROD_SCHEME"],
      ipa_name: ENV["PREPROD_IPA_NAME"],
      tryouts_app_id: ENV["PREPROD_TRYOUTS_APP_ID"],
      tryouts_api_token: ENV["PREPROD_TRYOUTS_API_TOKEN"],
      google_service_info_plist_path: ENV["PREPROD_GOOGLE_SERVICE_INFO_PLIST_PATH"]
    )

    tryouts_release = lane_context[SharedValues::TRYOUTS_BUILD_INFORMATION] 

    tryouts_preprod_release = {text: app_name_for_target(target: target), url: tryouts_release["download_url"]}

    if send_notification
      send_tryouts_target_notification_to_slack(target: target)
    end
  end

  lane :deploy_prod_app_to_tryouts do |options|
    send_notification = options[:send_notification] || true
    target = "Prod"
    app_id = ENV["APP_ID"]

    if app_id == nil
      UI.important "No app is found to deploy to Tryouts [#{target}]"
      next
    end

    if send_notification
      check_prod_env_variables
    end

    deploy_to_tryouts(
      target: target,
      app_identifier: app_id,
      scheme: ENV["PROD_SCHEME"],
      ipa_name: ENV["PROD_IPA_NAME"],
      tryouts_app_id: ENV["PROD_TRYOUTS_APP_ID"],
      tryouts_api_token: ENV["PROD_TRYOUTS_API_TOKEN"],
      google_service_info_plist_path: ENV["PROD_GOOGLE_SERVICE_INFO_PLIST_PATH"]
    )

    tryouts_release = lane_context[SharedValues::TRYOUTS_BUILD_INFORMATION] 

    tryouts_prod_release = {text: app_name_for_target(target: target), url: tryouts_release["download_url"]}

    if send_notification
      send_tryouts_target_notification_to_slack(target: target)
    end
  end

  private_lane :app_name_for_target do |options|
    ENV["APP_NAME"] + " " + options[:target]
  end

  private_lane :send_tryouts_target_notification_to_slack do |options|
    tryouts_release = lane_context[SharedValues::TRYOUTS_BUILD_INFORMATION] 

    app_name = app_name_for_target(target: options[:target])

    action = {
      text: app_name, 
      url: tryouts_release["download_url"]
    }

    send_tryouts_all_notification_to_slack(actions: [action])
  end

  private_lane :send_tryouts_all_notification_to_slack do |options|
    available_actions = options[:actions]
    actions = []

    for action in available_actions
      actions.push({type: "button", text: action.name, url: action.url})
    end

    if actions.empty?
      next
    end

    notify_slack_for_success(
      message: "🚀 App is deployed to Tryouts!",
      attachment_properties: {
        actions: actions
      }
    )
  end

  lane :deploy_all_apps_to_testflight do |options|
    deploy_staging_app_to_testflight
    deploy_preprod_app_to_testflight
    deploy_store_app_to_testflight
  end

  lane :deploy_staging_app_to_testflight do |options|
    check_staging_env_variables

    deploy_to_testflight(
      target: "Staging",
      app_identifier: ENV["STAGING_APP_ID"],
      scheme: ENV["STAGING_SCHEME"],
      ipa_name: ENV["STAGING_IPA_NAME"],
      export_method: "ad-hoc",
      google_service_info_plist_path: ENV["STAGING_GOOGLE_SERVICE_INFO_PLIST_PATH"]
    )
  end

  lane :deploy_preprod_app_to_testflight do |options|
    check_preprod_env_variables

    deploy_to_testflight(
      target: "Preprod",
      app_identifier: ENV["PREPROD_APP_ID"],
      scheme: ENV["PREPROD_SCHEME"],
      ipa_name: ENV["PREPROD_IPA_NAME"],
      export_method: "ad-hoc",
      google_service_info_plist_path: ENV["PREPROD_GOOGLE_SERVICE_INFO_PLIST_PATH"]
    )
  end

  lane :deploy_store_app_to_testflight do |options|
    check_store_env_variables

    deploy_to_testflight(
      target: "Store",
      app_identifier: ENV["APP_ID"],
      scheme: ENV["SCHEME"],
      ipa_name: ENV["IPA_NAME"],
      export_method: "app-store",
      google_service_info_plist_path: ENV["GOOGLE_SERVICE_INFO_PLIST_PATH"]
    )
  end

  lane :sync_all_dev_certs do |options|
    sync_staging_dev_cert
    sync_preprod_dev_cert
    sync_prod_dev_cert
  end

  lane :sync_staging_dev_cert do |options|
    sync_dev_cert(
      target: "Staging",
      app_identifier: ENV["STAGING_APP_ID"]
    )
  end

  lane :sync_preprod_dev_cert do |options|
    sync_dev_cert(
      target: "Preprod",
      app_identifier: ENV["PREPROD_APP_ID"]
    )
  end

  lane :sync_prod_dev_cert do |options|
    sync_dev_cert(
      target: "Prod",
      app_identifier: ENV["APP_ID"]
    )
  end

  error do |lane, exception, options|
    notify_slack_for_error(
      message: "🚑 Houston, we have a problem!",
      attachment_properties: {
        fields: [
          {
            title: "Git Tag",
            value: last_git_tag,
            short: true
          },
          {
            title: "Error",
            value: exception.to_s,
            short: false
          }
        ]
      }
    )
  end

  lane :deploy_to_tryouts do |options|
    target = options[:target]
    app_identifier = options[:app_identifier]

    if app_identifier == nil
      UI.important "No app is found to deploy to Tryouts [#{target}]"
      next
    end

    #1
    clean_build_artifacts

    #2
    register_connect_api_key

    #3
    tryouts_app_id = options[:tryouts_app_id]
    tryouts_api_token = options[:tryouts_api_token]

    register_missing_devices(
      tryouts_app_id: tryouts_app_id,
      tryouts_api_token: tryouts_api_token
    )

    #4
    sign(
      type: "adhoc",
      app_identifier: app_identifier
    )

    #5
    archive(
      configuration: ENV["ADHOC_BUILD_CONFIGURATION"],
      scheme: options[:scheme],
      output_name: options[:ipa_name],
      export_method: "ad-hoc",
      is_store: false
    )

    #6
    upload_to_tryouts(
      tryouts_app_id: tryouts_app_id,
      tryouts_api_token: tryouts_api_token
    )

    #7
    upload_symbols_to_crashlytics(gsp_path: options[:google_service_info_plist_path])
  end

  lane :deploy_to_testflight do |options|
    target = options[:target]
    app_identifier = options[:app_identifier]

    if app_identifier == nil
      UI.important "No app is found to deploy to Testflight [#{target}]"
      next
    end

    #1
    clean_build_artifacts

    #2
    register_connect_api_key

    #3
    sign(
      type: "appstore",
      app_identifier: app_identifier
    )

    #4
    archive(
        configuration: ENV["APP_STORE_BUILD_CONFIGURATION"],
        scheme: options[:scheme],
        output_name: options[:ipa_name],
        export_method: "app-store",
        is_store: true
    )

    #5
    upload_to_testflight(
      skip_submission: true, 
      skip_waiting_for_build_processing: true
    )

    #6
    upload_symbols_to_crashlytics(gsp_path: options[:google_service_info_plist_path])

    #7
    notify_slack_for_success(
      message: "🚀 App is deployed to TestFlight!",
      attachment_properties: {
        fields: [
          {
            title: "Git Tag",
            value: last_git_tag,
            short: true
          },
          {
            title: "Target",
            value: target,
            short: true
          }
        ]
      }
    )
  end

  # Register New Devices Taken From Tryouts
  lane :register_missing_devices do |options|
    connection = Faraday.new "https://api.tryouts.io/v1/applications/#{options[:tryouts_app_id]}/testers/" do |conn|
      conn.headers["Authorization"] = options[:tryouts_api_token]
      conn.request :url_encoded
      conn.response :json, :content_type => /\bjson$/
      conn.use FaradayMiddleware::FollowRedirects
      conn.adapter Faraday.default_adapter
    end

    response = connection.get
    results = response.body["results"]

    if results == nil
      next
    end

    for tester in results
      for device in tester["devices"]
        if device["os"] == "1" # iOS
          name = "#{tester["name"]} #{device["model"]}"
          udid = device["udid"]

          register_device(
            name: name,
            udid: udid
          )
          UI.message "#{name}: #{udid}"
        end
      end
    end
  end

  lane :sync_dev_cert do |options|
    target = options[:target]
    app_identifier = options[:app_identifier]

    if app_identifier == nil
      UI.important "No app is found to sync development certificates [#{target}]"
      next
    end

    register_connect_api_key

    sign(
      type: "development",
      app_identifier: app_identifier
    )
  end

  # Sign Cerificates For Given Profile Type
  lane :sign do |options|
    match(
      type: options[:type], 
      app_identifier: options[:app_identifier],
      force_for_new_devices: true,
      verbose: true
    )
  end

  #Register api key for app store connect 
  lane :register_connect_api_key do |options|
    app_store_connect_api_key(
      key_id: ENV["KEY_ID"],
      issuer_id: ENV["ISSUER_ID"],
      key_content: ENV["KEY_CONTENT"],
      in_house: ENV["IN_HOUSE"]
    )
  end

  lane :install_pods do |options|
    ENV["COCOAPODS_SCHEME"] = options[:is_store] ? "production" : "development"

    cocoapods(
      repo_update: false,
      clean_install: true
    )
  end

  lane :build do |options|
    #1
    install_pods(is_store: options[:is_store])

    #2
    scan(
      app_identifier: options[:app_identifier],
      scheme: options[:scheme],
      clean: true,
      build_for_testing: true
    )
  end

  lane :archive do |options|
    #1
    install_pods(is_store: options[:is_store])

    #2
    gym(
      workspace: ENV["WORKSPACE"],
      configuration: options[:configuration],
      scheme: options[:scheme],
      clean: true,
      output_directory: "./archive",
      output_name: options[:output_name],
      include_bitcode: true,
      xcargs: {:BITCODE_GENERATION_MODE => "bitcode"},
      export_method: options[:export_method],
      verbose: true
    )
  end

  lane :upload_to_tryouts do |options|
    tryouts(
      app_id: options[:tryouts_app_id],
      api_token: options[:tryouts_api_token],
      build_file: lane_context[SharedValues::IPA_OUTPUT_PATH],
      notify: 1,
      status: 2,
      notes_path: "./Release-Notes.md"
    )
  end

  lane :notify_slack_for_success do |options|
    notify_slack(
      default_payloads: [:git_branch],
      is_success: true,
      message: options[:message],
      attachment_properties: options[:attachment_properties]
    )
  end

  lane :notify_slack_for_error do |options|
    notify_slack(
      default_payloads: [:git_branch, :lane],
      is_success: false,
      message: options[:message],
      attachment_properties: options[:attachment_properties]
    )
  end

  lane :notify_slack do |options|
    slack_webhook_url = ENV["SLACK_WEBHOOK_URL"]

    if slack_webhook_url == nil
      UI.important "No 'Slack' webhook url is provided!"
      next
    end

    slack(
      slack_url: slack_webhook_url,
      default_payloads: options[:default_payloads],
      success: options[:is_success],
      message: options[:message],
      attachment_properties: options[:attachment_properties]
    )
  end

  lane :check_store_env_variables do |options|
    variables = shared_env_variables + [
      "APP_ID", 
      "SCHEME",
      "IPA_NAME",
      "GOOGLE_SERVICE_INFO_PLIST_PATH",
      "APP_STORE_BUILD_CONFIGURATION",
      "FASTLANE_ITC_TEAM_NAME",
    ]

    ensure_env_vars(
      env_vars: variables
    )
  end

  lane :check_prod_env_variables do |options|
    variables = shared_env_variables + [
      "APP_ID", 
      "PROD_SCHEME",
      "PROD_IPA_NAME",
      "PROD_TRYOUTS_APP_ID",
      "PROD_TRYOUTS_API_TOKEN",
      "PROD_GOOGLE_SERVICE_INFO_PLIST_PATH",
      "ADHOC_BUILD_CONFIGURATION"
    ]

    ensure_env_vars(
      env_vars: variables
    )
  end

  lane :check_staging_env_variables do |options|
    variables = shared_env_variables + [
      "STAGING_APP_ID", 
      "STAGING_SCHEME",
      "STAGING_IPA_NAME",
      "STAGING_TRYOUTS_APP_ID",
      "STAGING_TRYOUTS_API_TOKEN",
      "STAGING_GOOGLE_SERVICE_INFO_PLIST_PATH",
      "ADHOC_BUILD_CONFIGURATION"
    ]

    ensure_env_vars(
      env_vars: variables
    )
  end

  lane :check_preprod_env_variables do |options|
    variables = shared_env_variables + [
      "PREPROD_APP_ID", 
      "PREPROD_SCHEME",
      "PREPROD_IPA_NAME",
      "PREPROD_TRYOUTS_APP_ID",
      "PREPROD_TRYOUTS_API_TOKEN",
      "PREPROD_GOOGLE_SERVICE_INFO_PLIST_PATH",
      "ADHOC_BUILD_CONFIGURATION"
    ]

    ensure_env_vars(
      env_vars: variables
    )
  end
end