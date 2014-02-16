#!/usr/bin/env python2.7

'''

check_graphite
Copyright (C) 2014  Andreas Jansson

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.

'''

import sys
import argparse
import requests
try:
    import simplejson as json
except ImportError:
    import json

EXIT_OK         = 0
EXIT_WARNING    = 1
EXIT_CRITICAL   = 2
EXIT_UNKNOWN    = 3

def check(host, metric, warning_threshold, critical_threshold,
          invert, function, duration, user, password):

    if not host.startswith('http'):
        host = 'http://%s' % host

    if warning_threshold is not None and warning_threshold > critical_threshold:
        invert = True

    url = '%s/render' % host
    params = {
        'target': metric,
        'format': 'json',
        'from': '-%dmins' % duration,
    }
    if user is not None and password is not None:
        auth = (user, password)
    else:
        auth = None

    try:
        r = requests.get(url, params=params, auth=auth)
    except requests.exceptions.ConnectionError, e:
        critical('Failed to contact Graphite server', e.message)
    try:
        data = r.json()
    except json.JSONDecodeError, e:
        critical('Could not decode JSON', r.content)

    series = [x for x, t in data[0]['datapoints']]
    value = apply_function(series, function)

    message = '%s = %s (%s over %s minutes)' % (metric, value, function, duration)
    info = 'critical %s %s' % ('<' if invert else '>', critical_threshold)
    if warning is not None:
        info += '\nwarning %s %s' % ('<' if invert else '>', warning_threshold)

    if invert:
        if value < critical_threshold:
            critical(message, info)
        if value < warning_threshold:
            warning(message, info)
    else:
        if value > critical_threshold:
            critical(message, info)
        if value > warning_threshold:
            warning(message, info)

    ok(message)

def apply_function(series, function):
    if function == 'max':
        return max(series)
    if function == 'min':
        return min(series)
    if function == 'sum':
        return sum(series)
    if function == 'average':
        return sum(series) / float(len(series))

def critical(message, info=None):
    print 'CRITICAL %s' % message
    if info:
        print '\n%s' % info
    sys.exit(EXIT_CRITICAL)

def warning(message, info=None):
    print 'WARNING %s' % message
    if info:
        print '\n%s' % info
    sys.exit(EXIT_WARNING)

def ok(message, info=None):
    print 'OK %s' % message
    if info:
        print '\n%s' % info
    sys.exit(EXIT_OK)

if __name__ == '__main__':
    parser = argparse.ArgumentParser('Nagios check for Graphite metrics')
    parser.add_argument('--host', '-o', required=True, help='Graphite host, e.g. graphite.example.com, https://1.2.3.4, graphite01:8080')
    parser.add_argument('--metric', '-m', required=True, help='Metric to check, e.g. web01.processes.fork_rate')
    parser.add_argument('--critical', '-c', type=float, required=True, help='Critical threshold')
    parser.add_argument('--warning', '-w', type=float, help='Warning threshold')
    parser.add_argument('--invert', '-i', action='store_true', help='Invert the check so that an alert is triggered if the value falls below the threshold. Invert is implied if warning threshold > critical threshold')
    parser.add_argument('--duration', '-d', type=int, default=5, help='Number of minutes of data to aggregate')
    parser.add_argument('--function', '-f', default='average', choices=['average', 'max', 'min', 'sum'], help='The aggregation function to apply to the time series')
    parser.add_argument('--user', '-u', help='HTTP auth user')
    parser.add_argument('--password', '-p', help='HTTP auth password')

    args = parser.parse_args()
    check(host=args.host,
          metric=args.metric,
          warning_threshold=args.warning,
          critical_threshold=args.critical,
          invert=args.invert,
          function=args.function,
          duration=args.duration,
          user=args.user,
          password=args.password)
