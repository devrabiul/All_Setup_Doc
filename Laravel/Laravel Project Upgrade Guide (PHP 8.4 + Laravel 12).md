# Laravel Project Upgrade Guide (PHP 8.4 + Laravel 12)

এই ডকুমেন্টেশনটি আপনার Laravel প্রজেক্টকে **PHP 8.4** এবং **Laravel 12** এর সাথে কম্প্যাটিবল করার জন্য তৈরি করা হয়েছে। এটি প্রজেক্ট আপগ্রেড করার জন্য সব স্টেপ এবং গুরুত্বপূর্ণ চেকলিস্ট কভার করে।

---

## 📌 Step 0: PHP & Required Extensions চেক করুন

Laravel 12 এবং PHP 8.4 এর জন্য নিচের PHP extensions থাকা আবশ্যক:

```text
BCMath, Ctype, Fileinfo, JSON, Mbstring, OpenSSL, PDO, Tokenizer, XML, Curl
```

PHP version চেক করতে:

```bash
php -v
```

---

## 📌 Step 1: Composer.json আপডেট করুন

```json
"require": {
    "php": "^8.4",
    "laravel/framework": "^12.0"
}
```

তারপর রান করুন:

```bash
composer update
```

> ⚠️ নিশ্চিত করুন আপনার অন্যান্য প্যাকেজ Laravel 12 এবং PHP 8.4 এর সাথে কম্প্যাটিবল।

---

## 📌 Step 2: Deprecated PHP Functions রিপ্লেস করুন

PHP 8.4 এ অনেক ফাংশন বাদ দেওয়া বা ডিপ্রিকেটেড হয়েছে। প্রধান ফাংশন এবং বিকল্পগুলো নিচে দেওয়া হলো:

| ❌ Deprecated Function                           | ✅ Replacement Suggestion                                             |
| ----------------------------------------------- | -------------------------------------------------------------------- |
| `create_function()`                             | Anonymous function / arrow function → `fn() =>`                      |
| `get_magic_quotes_gpc()`                        | Remove usage (not needed)                                            |
| `get_magic_quotes_runtime()`                    | Remove usage                                                         |
| `set_magic_quotes_runtime()`                    | Remove usage                                                         |
| `money_format()`                                | Use `NumberFormatter` (Intl extension)                               |
| `each()`                                        | Use `foreach`                                                        |
| `mbstring.func_overload` (ini setting)          | Remove                                                               |
| `ldap_sort()`                                   | Use `usort()` on `ldap_get_entries()` result                         |
| `gzgetss()`                                     | Use `gzgets()` and manually strip tags                               |
| `fgetss()`                                      | Use `fgets()` and manually strip tags                                |
| `restore_include_path()`                        | Manually set include_path if needed                                  |
| `utf8_encode()` / `utf8_decode()`               | Use `mb_convert_encoding()`                                          |
| `getmypid()` (deprecated on some builds)        | Use `posix_getpid()`                                                 |
| Dynamic properties (e.g. `$obj->newProp`)       | Declare property inside class or use `__get` / `__set` magic methods |
| `implode($pieces, $glue)` (old order)           | Use `implode($glue, $pieces)`                                        |
| Deprecated curly braces array access: `$arr{0}` | Use square brackets: `$arr[0]`                                       |

> 💡 Tip: PHPCompatibility টুল দিয়ে পুরো প্রজেক্ট স্ক্যান করলে ডিপ্রিকেটেড ফাংশন এবং incompatibility গুলো সহজে চিহ্নিত করা যায়।

---

## 📌 Step 3: Laravel Helper & Rule আপডেট করুন

### 🔹 String & Array Helpers

| ❌ Deprecated Helper | ✅ Replacement       |
| ------------------- | ------------------- |
| `str_plural()`      | `Str::plural()`     |
| `str_singular()`    | `Str::singular()`   |
| `camel_case()`      | `Str::camel()`      |
| `snake_case()`      | `Str::snake()`      |
| `studly_case()`     | `Str::studly()`     |
| `str_contains()`    | `Str::contains()`   |
| `str_is()`          | `Str::is()`         |
| `str_random()`      | `Str::random()`     |
| `starts_with()`     | `Str::startsWith()` |
| `ends_with()`       | `Str::endsWith()`   |
| `array_first()`     | `Arr::first()`      |
| `array_last()`      | `Arr::last()`       |
| `array_except()`    | `Arr::except()`     |
| `array_only()`      | `Arr::only()`       |
| `array_pluck()`     | `Arr::pluck()`      |
| `array_get()`       | `Arr::get()`        |
| `array_set()`       | `Arr::set()`        |
| `array_has()`       | `Arr::has()`        |

### 🔹 Validation Rules

| ❌ Old Rule                   | ✅ New Rule                                    |
| ---------------------------- | --------------------------------------------- |
| `unique:users,email,'.$id`   | `Rule::unique('users', 'email')->ignore($id)` |
| `in_array:field`             | `Rule::in([ ... ])`                           |
| `not_in:value1,value2`       | `Rule::notIn([ ... ])`                        |
| `digits_between:min,max`     | `digits_between:min,max` (now stricter)       |
| `before:date` / `after:date` | `before_or_equal` / `after_or_equal`          |

### 🔹 Global Helpers (Deprecated)

| ❌ Deprecated         | ✅ Replacement           |
| -------------------- | ----------------------- |
| `dd()` (discouraged) | Use `dump()` or `ray()` |
| `config_path()`      | `app()->configPath()`   |
| `database_path()`    | `app()->databasePath()` |
| `public_path()`      | `app()->publicPath()`   |
| `storage_path()`     | `app()->storagePath()`  |
| `base_path()`        | `app()->basePath()`     |

---

## 📌 Step 4: PHP Compatibility Checker চালান

Composer দিয়ে ইন্সটল করুন:

```bash
composer require devrabiul/laravel-php-inspector --dev
```

Run:

```bash
php artisan phpcompat:check --php=8.4
```

> এটি পুরো প্রজেক্ট স্ক্যান করে দেখাবে কোন ফাংশন বা কোড **PHP 8.4 এ কাজ করবে না**।
> স্ক্যান রিপোর্টে দেখাবে:
>
> ```text
> 📄 Total files scanned: 871
> ❌ Total errors: 83
> ⚠️ Total warnings: 24
> ```

---

## 📌 Step 5: Third-party Package আপডেট করুন

সব প্যাকেজ চেক করতে:

```bash
composer outdated
```

Laravel 12 ও PHP 8.4 এর সাথে কম্প্যাটিবল ভার্সনে আপডেট করুন:

```bash
composer update vendor/package-name
```

> 💡 Tip: কিছু package PHP 8.4 compatible না হলে fork বা alternative package consider করুন।

---

## 📌 Step 6: Test চালান

Laravel Test চালান:

```bash
php artisan test
```

যদি টেস্ট না থাকে, **ম্যানুয়ালি ফিচারগুলো পরীক্ষা করুন**।
API, UI, এবং Console commands সব চেক করুন।

---

## 📌 Step 7: Deployment Checklist

* [ ] Local environment এ PHP 8.4 দিয়ে টেস্ট ✔️
* [ ] Deprecated function/হেল্পার রিপ্লেস ✔️
* [ ] Third-party প্যাকেজ আপডেট ✔️
* [ ] Unit test / Feature test পাস ✔️
* [ ] Staging সার্ভারে টেস্ট ✔️
* [ ] Production সার্ভারে ডেপ্লয় 🚀

---

✍️ **Note:**
Laravel টিমের অফিসিয়াল [Upgrade Guide](https://laravel.com/docs/12.x/upgrade) এবং [PHP 8.4 Migration Guide](https://www.php.net/releases/8.4/) অবশ্যই দেখে নেবেন।