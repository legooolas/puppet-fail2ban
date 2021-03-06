# Puppet module for fail2ban #

Install and manage fail2ban with puppet

To use this module just include the jail2ban class. To change default
configurations in /etc/fail2ban/jail.conf, you can pass values to parameters to
the fail2ban class. See section below for full list of parameters.

Here's an example that sets default ignored IP address to local host and
another non-routed IP:

~~~
class { 'fail2ban':
  ignoreip => '127.0.0.1 10.0.0.1',
}
~~~

You can create a jail with the fail2ban::jail defined type (see section below)
or you can use one of the predefined fail2ban::jail::* classes.

You can also create a filter for use with jails with the fail2ban::filter
defined type (see section below).

## Requirements ##

This module depends on the following modules to function:

 * puppetlabs' stdlib module (at least version 3.0.0)
 * puppetlabs' concat module (at least version 1.0.0)

## Parameters to fail2ban class ##

 * `ignoreip` Default ignored IP(s) when parsing logs. Default value is
   '127.0.0.1'. Multiple values should be separated by spaces
 * `bantime` Number of seconds during which reaching maxretry gets an IP
   banned. Default value is '600'
 * `maxretry` Number of times an IP address must trigger failgregexes to get
   banned. Default value is '3'
 * `backend` How should fail2ban look for modifications on log files. Default
   value is 'polling'
 * `destemail` Default email address that should get notifications with the
   actions that send emails. Default value is 'root@localhost'
 * `banaction` Default action to use for jails. Default value is
   'iptables-multiport'
 * `mta` Mail Transfer Agent program used for sending out email for actions
   that send out emails. Default value is 'sendmail'
 * `protocol` Default protocol for jails. Default value is 'tcp'
 * `action` Default action for jails. Default value is '%(action_)s', which is
   defined as '%(banaction)s[name=%(__name__)s, port="%(port)s",
   protocol="%(protocol)s]' in jail.conf.

## Defining jails ##

To define a jail, you can use one of the predefined jails (see list below). Or
you can define your own with the fail2ban::jail defined type:

~~~
fail2ban::jail { 'jenkins':
  port    => 'all',
  filter  => 'jenkins',
  logpath => '/var/log/jenkins.log',
}
~~~

Here's the full list of parameters you can use:

 * `port` List of port names, separated by commas, that will get blocked for a
   banned IP. Can be "all" to block all ports. This parameter is mandatory.
 * `filter` Name of the filter to use. This parameter is mandatory.
 * `logpath` Path of the log to monitor. This parameter is mandatory.
 * `ensure` Should this jail be present or not. Default value is present.
 * `enabled` Should this jail be enabled or not. The subtility between `ensure`
   and this parameter is that ensure will make the contents of the jail appear
   or disappear, while this parameter will let the jail contents be present in
   `jail.local` but the jail will be marked as disabled. Default value is
   'true'
 * `protocol` Override default protocol to ban ports for.
 * `maxretry` Override default number of trials that bans someone.
 * `findtime` Override default interval during which maxretry failures triggers
   a ban.
 * `action` Override default action used.
 * `banaction` Override default `banaction`. If you don't also override
   `action`, you will use the same default action template but with a different
   action name.
 * `bantime` Override default duration of a ban for an IP.
 * `ignoreip` Override default IP(s) to ignore (e.g. don't ban this IP).
 * `order` Optional numerical position. This lets you order jails as you see
   fit.

### Predefined jails ###

 * apache_noscript
 * apache_overflows
 * apache
 * asterisk
 * courierauth
 * couriersmtp
 * dovecot
 * dropbear
 * named_refused_tcp
 * pam_generic
 * postfix
 * proftpd
 * pure_ftpd
 * sasl
 * ssh_ddos
 * ssh
 * vsftpd
 * wuftpd
 * xinetd_fail

## Defining filters ##

You might want to define new filters for your new jails. To do that, you can
use the fail2ban::filter defined type:

~~~
fail2ban::filter { 'jenkins':
  failregexes => [
    # Those regexes are really arbitrary examples.
    'Invalid login to Jenkins by user mooh by IP \'<HOST>\'',
    'Forced entry trial by <HOST>',
  ],
}
~~~

Here's the full list of parameters you can use with the defined type:

 * `failregexes` List of regular expressions (strings) that, if matched, will
   increase IP's maxretry count. This parameter is mandatory.
 * `ensure` Should this filter be present or not. Default value is present
 * `ignoreregexes` List of regular expressions (strings) that, if matched, will
   invalidate failregex matching. Default value is an empty list.
 * `additional_defs` List of lines that could define more arbitrary values.
   Lines will be placed in the file as they are in the list. Default value is
   an empty list.

