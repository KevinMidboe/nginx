# Nginx config

Includes all nginx sites, snippets & required modules for homelab.

# Modules

Current configuration files require the following modules:
  - [headers-more-nginx-module](#headers-more-nginx-module)

We need to compile static modules like this from source (`modules-available/`) into `*.so` dynamic module files that can be loaded from `modules/`.[^1]


[^1]: [https://www.nginx.com/resources/wiki/extending/converting](https://www.nginx.com/resources/wiki/extending/converting)

## headers-more-nginx-module
  - [github.com/openresty/headers-more-nginx-module](https://github.com/openresty/headers-more-nginx-module)

We want to add headers from snippets, and in `server` & `location` blocks, through snippets and define in `location` blocks. Nginx `add_header` directive by design only adds headers if not previously added, this means `server` & snippet headers will be removed from response if `location` block uses `add_header`.

> There could be several add_header directives. These directives are inherited from the previous configuration level if and only if there are no add_header directives defined on the current level.[^2]

Instead we use module `headers-more` to replace directive `add_header 'key' 'value'` with `more_set_headers 'key value`. This will prevent removing previously defined headers and enable setting headers from snippets, and in `server` & `location` blocks.

[^2]: [https://nginx.org/en/docs/http/ngx_http_headers_module.html](https://nginx.org/en/docs/http/ngx_http_headers_module.html)

# CI/CD

[Drone CI config file](https://github.com/kevinmidboe/nginx/blob/main/.drone.yml) defines compiling modules from source and transfering to nginx host during deploy step.

**Setup environment:**
Current configs have references to SSL cert & key location. To validate nginx config with these references dummy cert & key files are created and symlinked to SSL cert & key locations defined in configs.

**Compiling nginx binary & modules:**
It is setup to pull git submodules and build `modules-available/` from source. This requires us to pull latest nginx source-code, configure it, build and folder reference in `--add-dynamic-module` flag when compiling nginx.

Since we compile nginx binary from source we need to enable flags for all common nginx modules we use in configs. View flags in [.circleci compile step](https://github.com/kevinmidboe/nginx/blob/main/.drone.yml). Flags currently set (prefix with `--with-{NGINX_MODULE_NAME}`):

 - [http_ssl_module](https://nginx.org/en/docs/http/ngx_http_ssl_module.html)
 - [http_v2_module](https://nginx.org/en/docs/http/ngx_http_v2_module.html)
 - [http_stub_status_module](https://nginx.org/en/docs/http/ngx_http_stub_status_module.html)
 - [http_gzip_static_module](https://nginx.org/en/docs/http/ngx_http_gzip_static_module.html)
 - [http_realip_module](https://nginx.org/en/docs/http/ngx_http_realip_module.html)

