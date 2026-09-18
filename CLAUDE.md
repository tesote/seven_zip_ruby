# tesote/seven_zip_ruby

> Project brain — established by dz-runner setup mode from this repo, then owned by
> its maintainers. Correct it in place; setup never overwrites a brain that exists.

SevenZipRuby — a Ruby gem (native C++ extension wrapping the official 7-Zip/p7zip sources) for reading, extracting and writing 7-Zip archives; maintained as an open-source library by masamitsu-murase (upstream) with CI on GitHub Actions.

## Stack

Ruby + Bundler (C++ native extension)

Toolchain: `mise.toml` — `bin/setup` runs `mise install`; the box carries only bun.

## Commands

| Task | Command |
|------|---------|
| setup | `bin/setup` — bundle install |
| dev | `bin/dev` — bundle exec rake build_local |
| check | `bin/check` — bundle exec rake build_local && bundle exec rspec spec/seven_zip_ruby_spec.rb |

## Structure

- `lib/seven_zip_ruby.rb` + `lib/seven_zip_ruby/*.rb` — the public Ruby API (SevenZipReader, SevenZipWriter, EntryInfo, ArchiveInfo, UpdateInfo, Exception). Loads the compiled binary `seven_zip_archive.so` from `lib/seven_zip_ruby/`.
- `lib/seven_zip_ruby/7z.dll`, `7z64.dll`, `7z.sfx`, `7zCon.sfx` — prebuilt official 7-Zip binaries shipped in the gem (Windows DLL + SFX stubs).
- `ext/seven_zip_ruby/` — the Ruby C++ extension glue (seven_zip_archive.cpp, utils.cpp, extconf.rb; per-platform `win32/mutex.h` / `posix/mutex.h`). Built by `rake build_local` via extconf + make; the `.so`/`.bundle` is copied into `lib/seven_zip_ruby/`.
- `ext/p7zip/` + `ext/CPP/` + `ext/C/` — vendored p7zip / 7-Zip C and C++ sources compiled into the extension. Never hand-edit.
- `spec/` — RSpec suite (`seven_zip_ruby_spec.rb` + helper) with fixture archives (`seven_zip.7z`, `seven_zip_password.7z`).
- `resources/` — logo and platform-gemspec template used by `rake build_platform`; excluded from the gem files.
- `Rakefile` — native build tasks: `build_local` (`build_binary` + `copy_binary`), `build_local_clean`, `build_platform`.
- Dependency rule: `spec/` → `lib/` (Ruby API) → compiled `seven_zip_archive.so` (from `ext/seven_zip_ruby/` + `ext/p7zip`). Changing `lib/` or `ext/` requires `rake build_local` before running specs.

## Conventions

- Ruby >= 2.0; gemspec declares dev deps only: bundler, rake, rspec (plus rubysl-* under rbx). No runtime gems.
- Tests are RSpec in `spec/`; the suite entry is `spec/seven_zip_ruby_spec.rb`, helper `spec/seven_zip_ruby_spec_helper.rb`; fixtures are real archives in `spec/*.7z` (incl. an encrypted one).
- Any change to `lib/` or `ext/seven_zip_ruby/` requires a rebuild before testing: `bundle exec rake build_local` (CI runs it as its install step).
- No RuboCop/lint config exists; style follows the existing code (`{ |i| ... }` braces, 2-space indent, snake_case).
- `ext/p7zip/**` is vendored 7-zip/p7zip source — never hand-edit it; changes belong in the glue code `ext/seven_zip_ruby/*.{cpp,h}`.
- Binary blobs in `lib/seven_zip_ruby/` (7z.dll, 7z64.dll, 7z.sfx, 7zCon.sfx, *.so, *.bundle) are build output/vendor, not source.
- Passwords are passed via the `password:` option to `Reader.open`/`extract_all`; wrong password must raise, not return bad data.
