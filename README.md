# Module 8A orm_practice — Set up instructions & troubleshooting notes
 
## What you'll need

- PHP 8.5+ and [Composer](https://getcomposer.org/)
- Laravel 13
- A local MySQL server (Homebrew MySQL or Herd) running on `127.0.0.1:3306`
- Git

## Setup instructions 

1. **Clone the repository**

```bash
git clone https://github.com/keani-julian/cs85-module8a-orm-practice.git
cd cs85-module8a-orm-practice
```

2. **Install PHP dependencies**
```bash
composer install
```

3. **Create your environment file**

```bash
cp .env.example .env
```

4. **Generate the application key** (fills in the blank `APP_KEY`)

```bash
php artisan key:generate
```

5. **Create the database** — connect to MySQL and create `orm_practice_db`:

```bash
mysql -h 127.0.0.1 -u root -e "CREATE DATABASE IF NOT EXISTS orm_practice_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

6. **Confirm the database settings in `.env`** match your MySQL server:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=orm_practice_db
DB_USERNAME=root
DB_PASSWORD=
```

7. **Run the migrations**

```bash
php artisan migrate
```

A successful run creates the default tables (`users`, `cache`, `jobs`, `sessions`, etc.) and confirms Laravel is connected to MySQL.

## Troubleshooting — what I actually ran into

My setup is macOS with Homebrew MySQL, so these are the exact problems I hit and how I had to work through them.

1. I couldn't connect as root. 

MySQL was already running, but `php artisan migrate` kept giving me `Access denied for user 'root'@'localhost'`. The assignment assumes a blank root password, but my root account (from an earlier module) had a password I didn't remember.

2. I then had to reset the root password using recovery mode.

I stopped MySQL, restarted it with
`mysqld_safe --skip-grant-tables` (which lets you in without a password), and connected with `mysql -u root`. Then I tried to blank the password:

```sql
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY '';
```

but the password policy blocked me.

`ALTER USER` failed with `Your password does not satisfy the current policy requirements` — MySQL's `validate_password` component won't allow an empty password. I then had to remove that component first, then set the blank password:

```sql
UNINSTALL COMPONENT 'file://component_validate_password';
ALTER USER 'root'@'localhost' IDENTIFIED BY '';
FLUSH PRIVILEGES;
```

3. After all the stopping/starting, MySQL still wasn't cleanly working. I got it running again by starting it directly with:

/opt/homebrew/opt/mysql/bin/mysqld_safe &

and after that `php artisan migrate` finally connected and created all the tables.
