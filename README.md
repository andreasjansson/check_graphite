check_graphite
==============

Super-basic Graphite check for Nagios. Will return CRITICAL if the Graphite
host is inaccessible. Can handle HTTP auth.

Usage:

    ./check_graphite        [-h] --host HOST --metric METRIC
                            --critical CRITICAL
                            [--warning WARNING] [--invert]
                            [--duration DURATION]
                            [--function {average,max,min,sum}]
                            [--user USER] [--password PASSWORD]
                            [--grafana {True,False}]
                            [--datasourceid DATASOURCEID]
                            [--authkey AUTHKEY]






    optional arguments:
        -h, --help              show this help message and exit
        --host HOST, -o HOST    Graphite host, e.g. graphite.example.com,
                                https://1.2.3.4, graphite01:8080
        --metric METRIC, -m METRIC
                                Metric to check, e.g. web01.processes.fork_rate
        --critical CRITICAL, -c CRITICAL
                                Critical threshold, e.g. 1.0, 3G
        --warning WARNING, -w WARNING
                                Warning threshold, e.g. -1.1k, 20
        --invert, -i            Invert the check so that an alert is triggered if the
                                value falls below the threshold. Invert is implied if
                                warning threshold > critical threshold
        --duration DURATION, -d DURATION
                                Number of minutes of data to aggregate
        --function {average,max,min,sum}, -f {average,max,min,sum}
                                The aggregation function to apply to the time series
        --user USER, -u USER  HTTP auth user
        --password PASSWORD, -p PASSWORD
                                HTTP auth password
        --grafana {True,False}, -g {True,False}
                                use grafana datasource proxy
        --datasourceid DATASOURCEID, -x DATASOURCEID
                                Grafana datasource id
        --authkey AUTHKEY, -a AUTHKEY
                                Grafana Authorization Key




Example Nagios commands:

    define command {
        command_name    check_graphite
        command_line    $USER1$/check_graphite --host graphite.mydomain:8008 --metric "$ARG1$" --critical $ARG2$ --warning $ARG3$ --user me --password changeme123 2>&1
    }

    define command {
        command_name    check_graphite_grafana
        command_line    $USER1$/check_graphite --host graphite.mydomain:8008 --metric "$ARG1$" --critical $ARG2$ --warning $ARG3$ --grafana --datasourceid 23 --authkey 12345ABSCD 2>&1
    }

Example Nagios services:

    define service {
        service_description    web01 load average
        host_name              web01.mydomain
        check_command          check_graphite!web01.load.load.shortterm!2.0!1.5
    }

    define service {
        service_description    web01 load average
        host_name              web01.mydomain
        check_command          check_graphite_grafana!web01.load.load.shortterm!2.0!1.5
    }

