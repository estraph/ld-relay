# Change log

All notable changes to the LaunchDarkly Relay will be documented in this file. This project adheres to [Semantic Versioning](http://semver.org).

## [5.8.0] - 2019-10-11
### Added:
- It is now possible to specify an infinite cache TTL for persistent feature stores by setting the cache TTL to a negative number, in which case the persistent store will never be read unless Relay restarts. See "Persistent storage" in `README.md`.

### Changed:
- When using a persistent store with an infinite cache TTL (see above), if Relay receives a feature flag update from LaunchDarkly and is unable to write it to the persistent store because of a database outage, it will still update the data in the in-memory cache so it will be available to the application. This is different from the existing behavior when there is a finite cache TTL: in that case, if the database update fails, the in-memory cache will _not_ be updated because the update would be lost as soon as the cache expires.
- When using a persistent store, if there is a database error (indicating that the database may be unavailable, or at least that the most recent update did not get persisted), Relay will continue to monitor the database availability. Once it returns to normal, if the cache TTL is finite, Relay will restart the LaunchDarkly connection to ensure that it receives and persists a full set of flag data; if the cache TTL is infinite, it will assume the cache is up to date and will simply write it to the database. See "Persistent storage" in `README.md`.


## [5.7.0] - 2019-09-18
### Added:
- The `exitAlways` configuration property (or `EXIT_ALWAYS`) variable causes the Relay Proxy to quit as soon as it has initialized all environments. This can be used for testing (just to make sure everything is working), or to perform a single poll of flags and put them into Redis or another database.
- The endpoints used by the PHP SDK (when it is not in LDD mode) were not previously supported. They are now. There is a new `ttlMinutes` property to configure caching behavior for PHP, similar to the TTL property on the LaunchDarkly dashboard. ([#68](https://github.com/launchdarkly/ld-relay/issues/68))
- The `logLevel` configuration properties (or `LOG_LEVEL` and `LD_LOG_LEVEL_EnvName`) allow you to request more or less verbose logging.
- If debug-level logging is enabled, the Relay Proxy will log every incoming HTTP request. ([#51](https://github.com/launchdarkly/ld-relay/issues/51))

### Changed:
- Log messages related to a specific environment are now prefixed with `[env: EnvironmentName]` (where `EnvironmentName` is the name you specified for that environment in your configuration) rather than `[LaunchDarkly Relay (SdkKey ending with xxxxx)]` (where `xxxxx` was the last 5 characters of the SDK key).

### Fixed:
- Updated Alpine base image in Docker build because some packages in the old image had vulnerabilities. (Thanks, [e96wic](https://github.com/launchdarkly/ld-relay/pull/74)!)
- Fixed a CI build problem for Go 1.8.

## [5.6.1] - 2019-08-05
### Fixed:
- Enabling TLS for Redis as a separate option was not working; it would only work if you specified a `rediss://` secure URL. Both methods now work. ([#71](https://github.com/launchdarkly/ld-relay/issues/71))

## [5.6.0] - 2019-08-02
### Added:
- You can now specify a proxy server URL in the configuration file, or in a Docker environment variable.
- Also, Relay now respects the standard Go environment variables `HTTP_PROXY` and `HTTPS_PROXY`.
- You may specify additional CA certificates for outgoing HTTPS connections.
- Relay now supports proxy servers that use NTLM authentication.
- You may specify a Redis password, or turn on TLS for Redis, without modifying the Redis URL.
- See `README.md` for details on configuring all of the above features.
 
### Changed:
- Extracted `Config` structs so that they could be configured programmatically when Relay is used as a library. (Thanks, [mightyguava](https://github.com/launchdarkly/ld-relay/pull/65)!)
 
### Fixed:
- The endpoints used by the client-side JavaScript SDKs were incorrectly returning _all_ flags, rather than only the flags with the "client-side" property (as the regular LaunchDarkly service does). ([#63](https://github.com/launchdarkly/ld-relay/issues/63))

## [5.5.2] - 2019-05-15
### Added
- Added documentation for the `REDIS_URL` environment variable to the README.
### Changed
- Replaced import paths for `gopkg.in/launchdarkly/go-client.v4` with `gopkg.in/launchdarkly/go-server-sdk.v4`. The newer import path reflects the new repository URL for the LaunchDarkly Server-side SDK for Go.

## [5.5.1] - 2018-12-19
### Fixed:
- When proxying events, the Relay now preserves information about what kind of platform they came from: server-side, mobile, or browser. Previously, it delivered all events to LaunchDarkly as if they were server-side events, which could cause your usage statistics to be wrong. ([#53](https://github.com/launchdarkly/ld-relay/issues/53))
- The `docker-entrypoint.sh` script, which is meant to run with any `sh`-compatible shell, contained some `bash` syntax that does not work in every shell.
- If the configuration is invalid because more than one database is selected (e.g., Redis + DynamoDB), the Relay will report an error. Previously, it picked one of the databases and ignored the others.
- Clarified documentation about persistent feature stores, and boolean environment variables.

## [5.5.0] - 2018-11-27
### Added
- The relay now supports additional database integrations: Consul and DynamoDB. As with the existing Redis integration, these can be used to store feature flags that will be read from the same database by a LaunchDarkly SDK client (as described [here](https://docs.launchdarkly.com/v2.0/docs/using-a-persistent-feature-store)), or simply as a persistence mechanism for the relay itself. See `README.MD` for configuration details.
- It is now possible to specify a Redis URL in `$REDIS_URL`, rather than just `$REDIS_HOST` and `$REDIS_PORT`, when running the Relay in Docker. (Thanks, [lukasmrtvy](https://github.com/launchdarkly/ld-relay/pull/48)!)

### Fixed
- When deployed in a Docker container, the relay no longer runs as the root user; instead, it creates a user called `ldr-user`. Also, the configuration file is now in `/ldr` instead of in `/etc`. (Thanks, [sdif](https://github.com/launchdarkly/ld-relay/pull/45)!)

## [5.4.4] - 2018-11-19

### Fixed
- Fixed a bug that would cause the relay to hang after the LaunchDarkly client finished initializing, if you had previously queried any relay endpoint _before_ the client finished initializing (e.g. if there was a delay due to network problems and you checked the status resource during that delay).

## [5.4.3] - 2018-09-97

### Added 
- Return flag evaluation reasons for mobile and client-side endpoints when `withReasons=true` query param is set.
- Added support for metric collection configuration in docker entrypoint script. Thanks @matthalbersma for the suggestion.

### Fixes
- Fix internal relay metrics event field.

## [5.4.2] - 2018-08-29

### Changed
- Use latest go client (4.3.0).
- Preserves the "client-side-enabled" property of feature flags so that the new SDK method AllFlagsState() - which has an option for selecting only client-side flags - will work correctly when using the relay.

### Fixes
- Ensure that server-side eval endpoints don't panic (work around gorilla mux bug)


## [5.4.1] - 2018-08-24

### Fixed
- Strip trailing slash that was breaking default urls for polling and streaming.
- If set X-LaunchDarkly-User-Agent will be used instead of the User-Agent header for metrics.
- Documentation improvements.

## [5.4.0] - 2018-08-09

### Added
- The Relay now supports the streaming endpoint used by current mobile SDKs. It does not fully proxy the stream, but will send an event causing mobile SDKs to re-request their flags when flags have changed.

## [5.3.1] - 2018-02-03

### Fixed

- Route metrics are now tagged correctly with the route name.
- Datadog exporter now works with 32-bit builds.


## [5.3.0] - 2018-07-30

### Added

- The Relay now supports exporting metrics and traces to Datadog, Prometheus, and Stackdriver. See the README for configuration instructions.

### Changed

- Packages intended for internal relay use are now marked as internal and no longer accessible exsternally.
- The package is now released using [goreleaser](https://goreleaser.com/).

## [5.2.0] - 2018-07-13

### Added
- The Redis server location-- and logical database-- can now optionally be specified as a URL (`url` property in `[redis]` section) rather than a hostname and port.
- The relay now sends usage metrics back to LaunchDarkly at 1-minute intervals.

### Fixed
- The /sdk/goals/<envId> endpoint now supports caching and repeats any headers it received to the client.

