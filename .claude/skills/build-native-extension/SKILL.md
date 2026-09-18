---
description: Rebuilding the C++ native extension via rake build_local after touching lib/ or ext/. Use for build_local, extconf, seven_zip_archive.so, compile error, native extension.
---

> House skill — established by dz-runner setup mode from this repo's own evidence, then owned by its maintainers. Correct it in place; setup never overwrites a skill that exists.

1. Run `bundle install` if not already done.
2. Build and copy the native binary: `bundle exec rake build_local`.
   - It runs extconf.rb inside `ext/seven_zip_ruby`, invokes make (override with `MAKE=...` env; nmake on mswin), and copies `seven_zip_archive.so|bundle` into `lib/seven_zip_ruby/`.
   - Compile errors in `ext/p7zip/**` -> stop; that is vendored source, do not patch it.
3. Clean rebuild when stale: `bundle exec rake build_local_clean && bundle exec rake build_local` (or `rake build_local_all`).
4. Prove the build: `bundle exec rspec spec/seven_zip_ruby_spec.rb`.
5. Only the `set multi_thread` example may legitimately fail from machine timing -> re-run before investigating.
6. Never commit the copied `lib/seven_zip_ruby/seven_zip_archive.so|bundle`; they are build output.
