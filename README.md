# Owasp ZAP with support for authentication
With the new [hook mechanism](https://github.com/zaproxy/zaproxy/issues/4925) in the ZAP Docker images authentication can implemented more easily compared to the original [implementation](https://github.com/ICTU/zap-baseline/blob/master/zap-baseline-custom.py).

# Docker

Please find the Docker container here:

https://hub.docker.com/r/ictu/zap2docker-weekly

# Examples

1. Running a passive scan with automatic authentication.
```
docker run --rm -v $(pwd):/zap/wrk/:rw -t ictu/zap2docker-weekly zap-baseline.py -I \
  -t https://demo.website.net \
  -r testreport.html \
   --hook=/zap/auth_hook.py \ 
  -z "auth.loginurl=https://demo.website.net/login/index.php \
      auth.username="admin" \
      auth.password="sandbox" \
      auth.auto=1"
```

2. Running a full scan (max 10 mins spider and max 60 min scanning) with manual authentication and including an additional URL in the scope.
```
docker run --rm -v $(pwd):/zap/wrk/:rw -t ictu/zap2docker-weekly zap-full-scan.py -I -m 10 -T 60 \
  -t https://demo.website.net \
  -r testreport.html \
   --hook=/zap/auth_hook.py \
  -z "auth.loginurl=https://demo.website.net/login/index.php \
      auth.username="admin" \
      auth.password="sandbox" \
      auth.username_field="j_username" \
      auth.password_field="j_password" \
      auth.submit_field="submit" \
      auth.exclude=".*logout.*,http://url.com/somepath.*"
      auth.include="https://api.website.net.*"
```

Note: exclude URL's are comma separated regular expressions. Examples:
```
.*logout.*,http://url.com/logout.*
```

Note: If this is a SPA application add the param: `-j`. This enables the AJAX Spider.
Note: 
`-m 60` limits the spider to 60 minutes. 
`-T 60` limits the scanner to 60 minutes.
Note: `-I` means do not return an errorcode if there are issues found.

For more info on the different scantypes and parameters take a look at: https://www.zaproxy.org/docs/docker/

# Extra parameters

```
auth.auto                 Automatically try to find the login fields (username, password, submit). Default True.
auth.loginurl             The URL to the login page. Required.
auth.username             A valid username. Required.
auth.password             A valid password. Required.
auth.username_field       The HTML name or id attribute of the username field.
auth.password_field       The HTML name or id attribute of the password field.
auth.submit_field         The HTML name or id attribute of the submit field.
auth.first_submit_field   The HTML name or id attribute of the first submit field (in case of username -> next page -> password -> submit).
auth.exclude              Comma separated list of excluded URL's (regex). Default: (logout|uitloggen|afmelden|signout)
auth.include              Comma separated list of included URL's (regex). Default: only the target URL and everything below it.
```

# Limitations
1. Since this authentication solution uses a webdriver a [custom image](https://hub.docker.com/repository/docker/ictu/zap2docker-weekly) is needed to meet these requirements.
2. Cookies that are automatically set by this script will not add flags like HttpOnly, Secure and SameSite. ZAP doest not support setting these cookies using the API. This will result in false-positives in the report regarding these flags.
