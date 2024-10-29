## metadata的配置

    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: ingress_http
          access_log:
          - name: envoy.access_loggers.stdout
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.access_loggers.stream.v3.StdoutAccessLog
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match:
                  prefix: "/"
                route:
                  cluster: nginx_cluster
                metadata:
                  filter_metadata:
                     my_custom_namespace:
                       authenticated: true
                       rate_limit: 100
          http_filters:
          - name: envoy.filters.http.rsa
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.wasm.v3.EnvironmentVariables
              host_env_keys:
                - "/auth/admin/realms/paas/enterpriseUser/updatePassword"
                - "/auth/admin/realms/paas/enterpriseUser/findUser"
                - "/auth/admin/realms/paas/api/organization/team/encryption/get"
                - "/auth/admin/realms/paas/api/organization/team/encryption/post/json"
                - "/auth/admin/realms/paas/api/organization/team/encryption/post/form-data"
                - "/auth/admin/realms/paas/api/organization/team/encryption/post/x-www-form-urlencoded"
              key_values:
                private_key_path: "/etc/envoy/keys/private.pem"
                public_key_path: "/etc/envoy/keys/public.pem"
          - name: envoy.filters.http.router
## 配置类代码

```
#include "envoy/config/filter/network/http_connection_manager/v3/http_connection_manager.pb.h"
#include "envoy/registry/registry.h"
#include "envoy/server/filter_config.h"

namespace Envoy {
namespace Extensions {
namespace HttpFilters {
namespace MyCustomFilter {

class MyCustomFilterConfig {
public:
  MyCustomFilterConfig(const envoy::config::filter::network::http_connection_manager::v3::HttpConnectionManager& config) {
    const auto& metadata = config.metadata().filter_metadata().find("my_custom_namespace");
    if (metadata != config.metadata().filter_metadata().end()) {
      // 解析 metadata
      const auto& feature_flag = metadata->second.fields().find("feature_flag");
      if (feature_flag != metadata->second.fields().end()) {
        feature_flag_ = feature_flag->second.bool_value();
      }
      // 其他解析逻辑
    }
  }

  bool featureFlag() const { return feature_flag_; }

private:
  bool feature_flag_{false};
};

// 注册过滤器等...

} // namespace MyCustomFilter
} // namespace HttpFilters
} // namespace Extensions
} // namespace Envoy

```