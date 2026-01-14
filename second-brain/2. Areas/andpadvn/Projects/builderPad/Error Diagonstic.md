

```
LoadError: libruby.so.3.2: cannot open shared object file: No such file or directory - /usr/local/bundle/gems/psych-5.1.2/lib/psych.so
  <internal:/usr/local/lib/ruby/site_ruby/3.3.0/rubygems/core_ext/kernel_require.rb>:136:in `require'
```

Already fixed: https://github.com/rubygems/rubygems/pull/8104
Already deployed: https://github.com/rubygems/rubygems/releases/tag/bundler-v2.5.22
How to fix in buildpad:
https://github.com/rubygems/rubygems/issues/8099#issuecomment-2404489765
remove those file 
```
docker compose run --rm --no-deps web rm -rf /usr/local/bundle/gems/psych-5.1.2/lib/psych.so /usr/local/bundle/gems/stringio-3.1.2/lib/stringio.so /usr/local/bundle/gems/date-3.4.1/lib/date_core.so /usr/local/bundle/gems/psych-5.2.1/lib/psych.so
```


Now I got error
```
Thanks for installing Vite Ruby!

If you upgraded the gem manually, please run:
        bundle exec vite upgrade
1 installed gem you directly depend on is looking for funding.
  Run `bundle fund` for details
yarn install v1.22.22
[1/4] Resolving packages...
[2/4] Fetching packages...
error Command failed.
Exit code: 128
Command: git
Arguments: clone https://github.com/88labs/andpad-viewer360.git /usr/local/share/.cache/yarn/v6/.tmp/32b1bc4c2db6e5bb751b3ca8dd1be262
Directory: /andpad
Output:
Cloning into '/usr/local/share/.cache/yarn/v6/.tmp/32b1bc4c2db6e5bb751b3ca8dd1be262'...
error: RPC failed; curl 92 HTTP/2 stream 7 was not closed cleanly: CANCEL (err 8)
error: 3582 bytes of body are still expected
fetch-pack: unexpected disconnect while reading sideband packet
fatal: early EOF
fatal: fetch-pack: invalid index-pack output
info Visit https://yarnpkg.com/en/docs/cli/install for documentation about this command.

```

I tried to run as the suggestion
```
bundle exec vite upgrade
```


Now I got this error when try to run rspec
docker compose exec web rspec ./spec/requests/api/internal/remote/clients/users/check_enable_id_whitelist_spec.rb

```
An error occurred while loading spec_helper.
Failure/Error: autoload :Faker, "#{Gem.loaded_specs['faker'].full_require_paths.first}/faker"

NoMethodError:
  undefined method `full_require_paths' for nil

```


show route
bundle exec rake routes