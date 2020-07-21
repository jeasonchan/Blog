# 1 前言
之前使用mysql都是直接安装mysql应用，然后在本地使用mysqld命令启动起来，其余的中间件也是采用类似的方式，这样就带来的一些问题，使用的物理机的PATH中的增加了很多bin的路径，而且启动的参数也各式各样的，于是想到在ubuntu中使用docker来运行一些中间件实例。

接下来从0开始讲解，在ubuntu中使用docker配配置、运行mysql实例。

# 2 安装、配置docker

## 2.1 apt install
直接使用：

```bash
sudo apt install dokcer.io
```


到现在还有很多教程在放屁，让安装docker ce……真的副了，人云亦云……很久之前，docker.io 由ubuntu团队维护，暂停维护了一段时间后，目前已经恢复正常维护了，且docker的版本和官方的docker ce版本基本一致，所以，现在建议直接在ubuntu中使用docker.io。

## 2.2  配置面免密使用docker命令
要使用的docker命令，一般都是必须使用root权限才能运行起来，在生产环境中无可非议，这样做能保证服务的安全，但是，自己的平时练习就么有必要了，因为：

1. 每次都要输入sudo很麻烦；如果，su root，docker会缺少命令补全功能……
2. vscode是以普通用户的身份启动的，里面的插件docker访问本机运行的docker当要也需要root权限，很显然以普通用户身份启动的vscode无法正常使用需要root权限的docker

解决方法就是，**将当前用户加入到docker用户组中**。

```bash
# 创建用户组 docker
sudo groupadd docker

# 打印一下当前的用户名称
echo $USER

# 经当前用户加入到docker组中
sudo usermod -aG docker $USER
```

重启系统即可，就能以普通用户的身份运行docker命令

## 2.3 配置自启动
众所周知，docker其实是一个b/s结构的本地应用。server端是运行在本地的，windows中叫服务（service），linux中叫守护进程（daemon），linux中手动启动一个守护进程的方式可以用：

```bash
# 启动
service docker start
# 停止
service docker stop
```

没有开机就启动的需求，每次要用的时候手动启动即可，要想开机启动可以通过systemctl命令实现开机对该程序的引导执行。

```bash
# 
$ sudo systemctl enable docker

# To disable this behavior, use disable instead.

$ sudo systemctl disable docker
```

## 2.4 优化镜像下载速度
经过之前的一些配置，守护进程已经启动的情况下，vscode的docker插件也应该是能正常使用了。

但是！使用dokcer pull去下载dockerHub中的公共镜像的时候，dockerHub默认的服务器在国外，下载会比较慢，建议切换成国内的云服务商提供的加速镜像地址。

1. 注册并登陆阿里云账号
2. 进入 https://cr.console.aliyun.com/cn-hangzhou/instances/repositories  阿里的容器镜像服务地址
3. 左下角"镜像加速"，已经自动生成了切换镜像的命令行：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://每个人都不一样的字符串.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

本质上就是创建了一个配置文件：/etc/docker/daemon.json，向里面写入了镜像地址，然后通过systemctl命令重启了守护进程


经过一番配置，下面就可以开始下载mysql镜像，并运行了

# 3 拉取、运行、配置mysql镜像

```bash
# latest可以省略
docker pull mysql:latest

# 罗列一下本地的镜像
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              36304d3b4540        9 days ago          104MB
mysql               latest              30f937e841c8        2 weeks ago         541MB

# 先尝试运行刚刚拉取的镜像，可以通过镜像名，也可以通过镜像ID
docker run 30f937e841c8
2020-06-07 15:03:22+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.20-1debian10 started.
2020-06-07 15:03:23+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2020-06-07 15:03:23+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.20-1debian10 started.
2020-06-07 15:03:23+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
	You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD
#报错了，每运行起来，docker ps也查不到其正在运行， 需要我们至少指明三个密码其中的一个

# -d 以后台方式运行，而不再占用当前的终端
# -p 容器内的端口映射到外部
# --name 指明实例名称
# -e 设置容器运行用到的环境变量
# 最后一个mysql是本地的镜像名称
docker run -d -p 3306:3306 --name mymysql -e MYSQL_ROOT_PASSWORD=123456 mysql

# 查看一下运行状态
docker stats

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
daf16e32d468        mymysql             0.75%               324.5MiB / 11.39GiB   2.78%               3.86kB / 0B         37MB / 471MB        38

# 使用DataGrip连接一下，已能正常使用

# 将当前可以用的mysql实例保存一下，供以后使用
docker commit daf16e32d468 mysql:latest_1

# 查看一下刚刚提交的镜像
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               latest_1            a314d06a41a5        3 minutes ago       541MB
redis               latest              36304d3b4540        9 days ago          104MB
mysql               latest              30f937e841c8        2 weeks ago         541MB

# 尝试运行一下
docker run -p 3306:3306   mysql:latest_1
2020-06-07 15:32:05+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.20-1debian10 started.
2020-06-07 15:32:05+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2020-06-07 15:32:05+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.20-1debian10 started.
2020-06-07 15:32:06+00:00 [Note] [Entrypoint]: Initializing database files
2020-06-07T15:32:06.024984Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
2020-06-07T15:32:06.025133Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.20) initializing of server in progress as process 42
2020-06-07T15:32:06.031952Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2020-06-07T15:32:07.225114Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2020-06-07T15:32:09.890345Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
2020-06-07 15:32:15+00:00 [Note] [Entrypoint]: Database files initialized
2020-06-07 15:32:15+00:00 [Note] [Entrypoint]: Starting temporary server
2020-06-07T15:32:15.370503Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
2020-06-07T15:32:15.370653Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.20) starting as process 89
2020-06-07T15:32:15.397683Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2020-06-07T15:32:15.683794Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2020-06-07T15:32:15.801544Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: '/var/run/mysqld/mysqlx.sock'
2020-06-07T15:32:16.047661Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2020-06-07T15:32:16.052051Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
2020-06-07T15:32:16.085802Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.20'  socket: '/var/run/mysqld/mysqld.sock'  port: 0  MySQL Community Server - GPL.
2020-06-07 15:32:16+00:00 [Note] [Entrypoint]: Temporary server started.
Warning: Unable to load '/usr/share/zoneinfo/iso3166.tab' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/leap-seconds.list' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/zone.tab' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/zone1970.tab' as time zone. Skipping it.

2020-06-07 15:32:18+00:00 [Note] [Entrypoint]: Stopping temporary server
2020-06-07T15:32:19.006840Z 10 [System] [MY-013172] [Server] Received SHUTDOWN from user root. Shutting down mysqld (Version: 8.0.20).
2020-06-07T15:32:21.738005Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.20)  MySQL Community Server - GPL.
2020-06-07 15:32:22+00:00 [Note] [Entrypoint]: Temporary server stopped

2020-06-07 15:32:22+00:00 [Note] [Entrypoint]: MySQL init process done. Ready for start up.

2020-06-07T15:32:22.262917Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
2020-06-07T15:32:22.263060Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.20) starting as process 1
2020-06-07T15:32:22.274919Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2020-06-07T15:32:22.561633Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2020-06-07T15:32:22.689342Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: '/var/run/mysqld/mysqlx.sock' bind-address: '::' port: 33060
2020-06-07T15:32:22.824900Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2020-06-07T15:32:22.829876Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
2020-06-07T15:32:22.858027Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.20'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.


# 已能成功运行，DataGrip也能正常访问数据，可见root用户的密码已经持久化到镜像中

```

可见，容器内的mysql运行时是可以直接从系统环境变量中取值的，个人推测应该是mysql的启动脚本做了一写检查判断。再结合之前的mysql使用经验，想改变容器内的nysql的实例参数，**个人猜测**至少有以下的方法：

1. 设置容器的环境变量
2. 直接编写容器内的配置文件
3. 将本地的配置文件映射到容器中，容器内的mysql从路径读取配置文件时就会使用本地的配置文件

接下来探索一下通过修改容器内的配置文件实现mysql配置。

## 3.1 编写容器内配置文件优化内存占用

首先要看一下mysql的启动文件是怎么写的，看看它是从哪些路径查找配置文件的。

```bash
# 以阻塞式的sh作为docker运行的主线程，
# 并且使用-it参数进入交互式终端
sudo docker run -it mysql sh

ls

bin  boot  dev	docker-entrypoint-initdb.d  entrypoint.sh  etc	home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

cat entrypoint.sh

# 输出以下内容：

#!/bin/bash
set -eo pipefail
shopt -s nullglob

# logging functions
mysql_log() {
	local type="$1"; shift
	printf '%s [%s] [Entrypoint]: %s\n' "$(date --rfc-3339=seconds)" "$type" "$*"
}
mysql_note() {
	mysql_log Note "$@"
}
mysql_warn() {
	mysql_log Warn "$@" >&2
}
mysql_error() {
	mysql_log ERROR "$@" >&2
	exit 1
}

# usage: file_env VAR [DEFAULT]
#    ie: file_env 'XYZ_DB_PASSWORD' 'example'
# (will allow for "$XYZ_DB_PASSWORD_FILE" to fill in the value of
#  "$XYZ_DB_PASSWORD" from a file, especially for Docker's secrets feature)
file_env() {
	local var="$1"
	local fileVar="${var}_FILE"
	local def="${2:-}"
	if [ "${!var:-}" ] && [ "${!fileVar:-}" ]; then
		mysql_error "Both $var and $fileVar are set (but are exclusive)"
	fi
	local val="$def"
	if [ "${!var:-}" ]; then
		val="${!var}"
	elif [ "${!fileVar:-}" ]; then
		val="$(< "${!fileVar}")"
	fi
	export "$var"="$val"
	unset "$fileVar"
}

# check to see if this file is being run or sourced from another script
_is_sourced() {
	# https://unix.stackexchange.com/a/215279
	[ "${#FUNCNAME[@]}" -ge 2 ] \
		&& [ "${FUNCNAME[0]}" = '_is_sourced' ] \
		&& [ "${FUNCNAME[1]}" = 'source' ]
}

# usage: docker_process_init_files [file [file [...]]]
#    ie: docker_process_init_files /always-initdb.d/*
# process initializer files, based on file extensions
docker_process_init_files() {
	# mysql here for backwards compatibility "${mysql[@]}"
	mysql=( docker_process_sql )

	echo
	local f
	for f; do
		case "$f" in
			*.sh)
				# https://github.com/docker-library/postgres/issues/450#issuecomment-393167936
				# https://github.com/docker-library/postgres/pull/452
				if [ -x "$f" ]; then
					mysql_note "$0: running $f"
					"$f"
				else
					mysql_note "$0: sourcing $f"
					. "$f"
				fi
				;;
			*.sql)    mysql_note "$0: running $f"; docker_process_sql < "$f"; echo ;;
			*.sql.gz) mysql_note "$0: running $f"; gunzip -c "$f" | docker_process_sql; echo ;;
			*.sql.xz) mysql_note "$0: running $f"; xzcat "$f" | docker_process_sql; echo ;;
			*)        mysql_warn "$0: ignoring $f" ;;
		esac
		echo
	done
}

mysql_check_config() {
	local toRun=( "$@" --verbose --help ) errors
	if ! errors="$("${toRun[@]}" 2>&1 >/dev/null)"; then
		mysql_error $'mysqld failed while attempting to check config\n\tcommand was: '"${toRun[*]}"$'\n\t'"$errors"
	fi
}

# Fetch value from server config
# We use mysqld --verbose --help instead of my_print_defaults because the
# latter only show values present in config files, and not server defaults
mysql_get_config() {
	local conf="$1"; shift
	"$@" --verbose --help --log-bin-index="$(mktemp -u)" 2>/dev/null \
		| awk -v conf="$conf" '$1 == conf && /^[^ \t]/ { sub(/^[^ \t]+[ \t]+/, ""); print; exit }'
	# match "datadir      /some/path with/spaces in/it here" but not "--xyz=abc\n     datadir (xyz)"
}

# Do a temporary startup of the MySQL server, for init purposes
docker_temp_server_start() {
	if [ "${MYSQL_MAJOR}" = '5.6' ] || [ "${MYSQL_MAJOR}" = '5.7' ]; then
		"$@" --skip-networking --socket="${SOCKET}" &
		mysql_note "Waiting for server startup"
		local i
		for i in {30..0}; do
			# only use the root password if the database has already been initializaed
			# so that it won't try to fill in a password file when it hasn't been set yet
			extraArgs=()
			if [ -z "$DATABASE_ALREADY_EXISTS" ]; then
				extraArgs+=( '--dont-use-mysql-root-password' )
			fi
			if docker_process_sql "${extraArgs[@]}" --database=mysql <<<'SELECT 1' &> /dev/null; then
				break
			fi
			sleep 1
		done
		if [ "$i" = 0 ]; then
			mysql_error "Unable to start server."
		fi
	else
		# For 5.7+ the server is ready for use as soon as startup command unblocks
		if ! "$@" --daemonize --skip-networking --socket="${SOCKET}"; then
			mysql_error "Unable to start server."
		fi
	fi
}

# Stop the server. When using a local socket file mysqladmin will block until
# the shutdown is complete.
docker_temp_server_stop() {
	if ! mysqladmin --defaults-extra-file=<( _mysql_passfile ) shutdown -uroot --socket="${SOCKET}"; then
		mysql_error "Unable to shut down server."
	fi
}

# Verify that the minimally required password settings are set for new databases.
docker_verify_minimum_env() {
	if [ -z "$MYSQL_ROOT_PASSWORD" -a -z "$MYSQL_ALLOW_EMPTY_PASSWORD" -a -z "$MYSQL_RANDOM_ROOT_PASSWORD" ]; then
		mysql_error $'Database is uninitialized and password option is not specified\n\tYou need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD'
	fi
}

# creates folders for the database
# also ensures permission for user mysql of run as root
docker_create_db_directories() {
	local user; user="$(id -u)"

	# TODO other directories that are used by default? like /var/lib/mysql-files
	# see https://github.com/docker-library/mysql/issues/562
	mkdir -p "$DATADIR"

	if [ "$user" = "0" ]; then
		# this will cause less disk access than `chown -R`
		find "$DATADIR" \! -user mysql -exec chown mysql '{}' +
	fi
}

# initializes the database directory
docker_init_database_dir() {
	mysql_note "Initializing database files"
	if [ "$MYSQL_MAJOR" = '5.6' ]; then
		mysql_install_db --datadir="$DATADIR" --rpm --keep-my-cnf "${@:2}"
	else
		"$@" --initialize-insecure
	fi
	mysql_note "Database files initialized"

	if command -v mysql_ssl_rsa_setup > /dev/null && [ ! -e "$DATADIR/server-key.pem" ]; then
		# https://github.com/mysql/mysql-server/blob/23032807537d8dd8ee4ec1c4d40f0633cd4e12f9/packaging/deb-in/extra/mysql-systemd-start#L81-L84
		mysql_note "Initializing certificates"
		mysql_ssl_rsa_setup --datadir="$DATADIR"
		mysql_note "Certificates initialized"
	fi
}

# Loads various settings that are used elsewhere in the script
# This should be called after mysql_check_config, but before any other functions
docker_setup_env() {
	# Get config
	declare -g DATADIR SOCKET
	DATADIR="$(mysql_get_config 'datadir' "$@")"
	SOCKET="$(mysql_get_config 'socket' "$@")"

	# Initialize values that might be stored in a file
	file_env 'MYSQL_ROOT_HOST' '%'
	file_env 'MYSQL_DATABASE'
	file_env 'MYSQL_USER'
	file_env 'MYSQL_PASSWORD'
	file_env 'MYSQL_ROOT_PASSWORD'

	declare -g DATABASE_ALREADY_EXISTS
	if [ -d "$DATADIR/mysql" ]; then
		DATABASE_ALREADY_EXISTS='true'
	fi
}

# Execute sql script, passed via stdin
# usage: docker_process_sql [--dont-use-mysql-root-password] [mysql-cli-args]
#    ie: docker_process_sql --database=mydb <<<'INSERT ...'
#    ie: docker_process_sql --dont-use-mysql-root-password --database=mydb <my-file.sql
docker_process_sql() {
	passfileArgs=()
	if [ '--dont-use-mysql-root-password' = "$1" ]; then
		passfileArgs+=( "$1" )
		shift
	fi
	# args sent in can override this db, since they will be later in the command
	if [ -n "$MYSQL_DATABASE" ]; then
		set -- --database="$MYSQL_DATABASE" "$@"
	fi

	mysql --defaults-extra-file=<( _mysql_passfile "${passfileArgs[@]}") --protocol=socket -uroot -hlocalhost --socket="${SOCKET}" "$@"
}

# Initializes database with timezone info and root password, plus optional extra db/user
docker_setup_db() {
	# Load timezone info into database
	if [ -z "$MYSQL_INITDB_SKIP_TZINFO" ]; then
		# sed is for https://bugs.mysql.com/bug.php?id=20545
		mysql_tzinfo_to_sql /usr/share/zoneinfo \
			| sed 's/Local time zone must be set--see zic manual page/FCTY/' \
			| docker_process_sql --dont-use-mysql-root-password --database=mysql
			# tell docker_process_sql to not use MYSQL_ROOT_PASSWORD since it is not set yet
	fi
	# Generate random root password
	if [ -n "$MYSQL_RANDOM_ROOT_PASSWORD" ]; then
		export MYSQL_ROOT_PASSWORD="$(pwgen -1 32)"
		mysql_note "GENERATED ROOT PASSWORD: $MYSQL_ROOT_PASSWORD"
	fi
	# Sets root password and creates root users for non-localhost hosts
	local rootCreate=
	# default root to listen for connections from anywhere
	if [ -n "$MYSQL_ROOT_HOST" ] && [ "$MYSQL_ROOT_HOST" != 'localhost' ]; then
		# no, we don't care if read finds a terminating character in this heredoc
		# https://unix.stackexchange.com/questions/265149/why-is-set-o-errexit-breaking-this-read-heredoc-expression/265151#265151
		read -r -d '' rootCreate <<-EOSQL || true
			CREATE USER 'root'@'${MYSQL_ROOT_HOST}' IDENTIFIED BY '${MYSQL_ROOT_PASSWORD}' ;
			GRANT ALL ON *.* TO 'root'@'${MYSQL_ROOT_HOST}' WITH GRANT OPTION ;
		EOSQL
	fi

	local passwordSet=
	if [ "$MYSQL_MAJOR" = '5.6' ]; then
		# no, we don't care if read finds a terminating character in this heredoc (see above)
		read -r -d '' passwordSet <<-EOSQL || true
			DELETE FROM mysql.user WHERE user NOT IN ('mysql.sys', 'mysqlxsys', 'root') OR host NOT IN ('localhost') ;
			SET PASSWORD FOR 'root'@'localhost'=PASSWORD('${MYSQL_ROOT_PASSWORD}') ;

			-- 5.5: https://github.com/mysql/mysql-server/blob/e48d775c6f066add457fa8cfb2ebc4d5ff0c7613/scripts/mysql_secure_installation.sh#L192-L210
			-- 5.6: https://github.com/mysql/mysql-server/blob/06bc670db0c0e45b3ea11409382a5c315961f682/scripts/mysql_secure_installation.sh#L218-L236
			-- 5.7: https://github.com/mysql/mysql-server/blob/913071c0b16cc03e703308250d795bc381627e37/client/mysql_secure_installation.cc#L792-L818
			-- 8.0: https://github.com/mysql/mysql-server/blob/b93c1661d689c8b7decc7563ba15f6ed140a4eb6/client/mysql_secure_installation.cc#L726-L749
			DELETE FROM mysql.db WHERE Db='test' OR Db='test\_%' ;
			-- https://github.com/docker-library/mysql/pull/479#issuecomment-414561272 ("This is only needed for 5.5 and 5.6")
		EOSQL
	else
		# no, we don't care if read finds a terminating character in this heredoc (see above)
		read -r -d '' passwordSet <<-EOSQL || true
			ALTER USER 'root'@'localhost' IDENTIFIED BY '${MYSQL_ROOT_PASSWORD}' ;
		EOSQL
	fi

	# tell docker_process_sql to not use MYSQL_ROOT_PASSWORD since it is just now being set
	docker_process_sql --dont-use-mysql-root-password --database=mysql <<-EOSQL
		-- What's done in this file shouldn't be replicated
		--  or products like mysql-fabric won't work
		SET @@SESSION.SQL_LOG_BIN=0;

		${passwordSet}
		GRANT ALL ON *.* TO 'root'@'localhost' WITH GRANT OPTION ;
		FLUSH PRIVILEGES ;
		${rootCreate}
		DROP DATABASE IF EXISTS test ;
	EOSQL

	# Creates a custom database and user if specified
	if [ -n "$MYSQL_DATABASE" ]; then
		mysql_note "Creating database ${MYSQL_DATABASE}"
		docker_process_sql --database=mysql <<<"CREATE DATABASE IF NOT EXISTS \`$MYSQL_DATABASE\` ;"
	fi

	if [ -n "$MYSQL_USER" ] && [ -n "$MYSQL_PASSWORD" ]; then
		mysql_note "Creating user ${MYSQL_USER}"
		docker_process_sql --database=mysql <<<"CREATE USER '$MYSQL_USER'@'%' IDENTIFIED BY '$MYSQL_PASSWORD' ;"

		if [ -n "$MYSQL_DATABASE" ]; then
			mysql_note "Giving user ${MYSQL_USER} access to schema ${MYSQL_DATABASE}"
			docker_process_sql --database=mysql <<<"GRANT ALL ON \`${MYSQL_DATABASE//_/\\_}\`.* TO '$MYSQL_USER'@'%' ;"
		fi

		docker_process_sql --database=mysql <<<"FLUSH PRIVILEGES ;"
	fi
}

_mysql_passfile() {
	# echo the password to the "file" the client uses
	# the client command will use process substitution to create a file on the fly
	# ie: --defaults-extra-file=<( _mysql_passfile )
	if [ '--dont-use-mysql-root-password' != "$1" ] && [ -n "$MYSQL_ROOT_PASSWORD" ]; then
		cat <<-EOF
			[client]
			password="${MYSQL_ROOT_PASSWORD}"
		EOF
	fi
}

# Mark root user as expired so the password must be changed before anything
# else can be done (only supported for 5.6+)
mysql_expire_root_user() {
	if [ -n "$MYSQL_ONETIME_PASSWORD" ]; then
		docker_process_sql --database=mysql <<-EOSQL
			ALTER USER 'root'@'%' PASSWORD EXPIRE;
		EOSQL
	fi
}

# check arguments for an option that would cause mysqld to stop
# return true if there is one
_mysql_want_help() {
	local arg
	for arg; do
		case "$arg" in
			-'?'|--help|--print-defaults|-V|--version)
				return 0
				;;
		esac
	done
	return 1
}

_main() {
	# if command starts with an option, prepend mysqld
	if [ "${1:0:1}" = '-' ]; then
		set -- mysqld "$@"
	fi

	# skip setup if they aren't running mysqld or want an option that stops mysqld
	if [ "$1" = 'mysqld' ] && ! _mysql_want_help "$@"; then
		mysql_note "Entrypoint script for MySQL Server ${MYSQL_VERSION} started."

		mysql_check_config "$@"
		# Load various environment variables
		docker_setup_env "$@"
		docker_create_db_directories

		# If container is started as root user, restart as dedicated mysql user
		if [ "$(id -u)" = "0" ]; then
			mysql_note "Switching to dedicated user 'mysql'"
			exec gosu mysql "$BASH_SOURCE" "$@"
		fi

		# there's no database, so it needs to be initialized
		if [ -z "$DATABASE_ALREADY_EXISTS" ]; then
			docker_verify_minimum_env

			# check dir permissions to reduce likelihood of half-initialized database
			ls /docker-entrypoint-initdb.d/ > /dev/null

			docker_init_database_dir "$@"

			mysql_note "Starting temporary server"
			docker_temp_server_start "$@"
			mysql_note "Temporary server started."

			docker_setup_db
			docker_process_init_files /docker-entrypoint-initdb.d/*

			mysql_expire_root_user

			mysql_note "Stopping temporary server"
			docker_temp_server_stop
			mysql_note "Temporary server stopped"

			echo
			mysql_note "MySQL init process done. Ready for start up."
			echo
		fi
	fi
	exec "$@"
}

# If we are sourced from elsewhere, don't perform any further actions
if ! _is_sourced; then
	_main "$@"
fi

# 进入配置文件夹
# conf文件，后面还带了个d，十有八九就是mysql的 daemon 的配置
cd /etc/mysql/conf.d

ls

# 有两个配置文件
docker.cnf  mysql.cnf

# 分别查看了一下
# 一个是[mysql]，一个是[mysqld]的
# 果断是应该修改[mysqld]文件


```

可见！！！**只需要修改容器内的  /etc/mysql/conf.d/docker.cnf ，并将修改commit到镜像中，即可每次使用修改后的配置运行**

接下来需要优化mysql配置，降低系统占用。

### 3.1.1 mysql的最低占用配置

```	
key_buffer = 16K
max_allowed_packet = 1M
thread_stack = 64K
table_cache = 4
sort_buffer = 64K
net_buffer_length = 2K
```

但是，自己本机开发时，默认配置的mysql的占用并不高，甚至都都没有空白页的chrome浏览器高，大概400m左右……


## 3.2 本地配置文件映射到容器中

将本地文件、文件夹映射到容器中，在容器内访问被映射的文件时就会直接访物理机里的文件路径，从而实现配置文件在物理机中，能被容器内的应用mysql直接读取。路径映射及需要映射的文件如下：

```bash
docker run --name my_mysql -p 3306:3306 -v ~/conf:/etc/mysql/conf.d -v ~/logs:/logs -v ~/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql

```



# 4 总结

最后，建议使用在本地创建配置文件，并通过docker命令映射到容器中供容器内的程序使用，如：

```bash
docker run --name my_mysql -p 3306:3306 -v ~/conf:/etc/mysql/conf.d -v ~/logs:/logs -v ~/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql

```
