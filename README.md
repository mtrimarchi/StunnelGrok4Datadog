# stunnel service log to Datadog using Grok Parser

This is a simple example used to extract informations from stunnel service log lines. Just create a new [pipeline](https://app.datadoghq.com/logs/pipelines) filtering out `service:stunnel` and the host where your stunnel instance is working on (i.e. `host:my-server`).

## Grok Parsing Rules

Create a **Grok Parser** and add these lines under *Define 1 or multiple parsing rules* box:

```
stunnel.service.connected_remote_server_from ^%{_date_stunnel} LOG%{_log_status}\[%{_session_id}\]\: Service \[%{_service_name}\] connected remote server from %{_local_ip}\:%{_local_port}$

stunnel.service.accepted_connection_from ^%{_date_stunnel} LOG%{_log_status}\[%{_session_id}\]\: Service \[%{_service_name}\] accepted connection from %{_client_ip}\:%{_client_port}$

stunnel.certificate.accepted ^%{_date_stunnel} LOG%{_log_status}\[%{_session_id}\]\: Certificate accepted at depth\=%{_cert_depth}\: %{_cert_info}$

stunnel.connection.closed ^%{_date_stunnel} LOG%{_log_status}\[%{_session_id}\]\: Connection closed\: %{_byte_sent_to_ssl} byte\(s\) sent to SSL\, %{_byte_sent_to_socket} byte\(s\) sent to socket$

stunnel.connection.reset ^%{_date_stunnel} LOG%{_log_status}\[%{_session_id}\]\: Connection reset\: %{_byte_sent_to_ssl} byte\(s\) sent to SSL\, %{_byte_sent_to_socket} byte\(s\) sent to socket$

stunnel.s_connect.connected ^%{_date_stunnel} LOG%{_log_status}\[%{_session_id}\]\: s_connect\: connected %{_backend_ip}\:%{_backend_port}$

stunnel.s_connect.connecting ^%{_date_stunnel} LOG%{_log_status}\[%{_session_id}\]\: s_connect\: connecting %{_backend_ip}\:%{_backend_port}$

stunnel.s_connect.no_route_to_host ^%{_date_stunnel} LOG%{_log_status}\[%{_session_id}\]\: s_connect\: connect %{_backend_ip}\:%{_backend_port}\: %{_error_message}$

stunnel.s_connect.connection_refused_by_peer ^%{_date_stunnel} LOG%{_log_status}\[%{_session_id}\]\: s_connect\: connect %{_backend_ip}\:%{_backend_port}\: %{_error_message}$

stunnel.s_connect.timeoutconnect_exceeded ^%{_date_stunnel} LOG%{_log_status}\[%{_session_id}\]\: s_connect\: s_poll_wait %{_backend_ip}\:%{_backend_port}\: %{_error_message}$

stunnel.general.error_message ^%{_date_stunnel} LOG%{_log_status}\[%{_session_id}\]\: %{_error_message}$
```

## Helper Rules

Open up **Advanced Settings**, on *Extract from:* declare `message` and under *Helper Rules:* box add these helper rules:

```
_backend_ip  %{ipv4:network.backend.ip}
_backend_port %{port:network.backend.port}
_byte_sent_to_socket %{integer:network.byte.sent_to_socket}
_byte_sent_to_ssl %{integer:network.byte.sent_to_ssl}
_cert_depth %{integer:stunnel.certificate.depth}
_cert_info %{data:stunnel.certificate.info}
_client_ip %{ipv4:network.client.ip}
_client_port %{port:network.client.port}$
_date_stunnel %{date("yyyy.MM.dd HH:mm:ss","Europe/Rome"):date}
_local_ip %{ipv4:network.local.ip}
_local_port %{port:network.local.port}
_log_status %{integer:stunnel.log_status}
_session_id %{data:stunnel.session_id}
_service_name %{data:stunnel.service_name}
_error_message %{regex("(No route to host|Connection refused|TIMEOUT|readsocket|writesocket|transfer|SSL|s_connect).*"):stunnel.error_message}
```

## Hints
If you need other info or resources check out this list of links
- [Datadog Processing](https://docs.datadoghq.com/logs/processing/)
- [Datadog Parsing](https://docs.datadoghq.com/logs/processing/parsing/)
- [Datadog Attributes Naming Convention](https://docs.datadoghq.com/logs/processing/attributes_naming_convention/)
- [Datadog Log Parsing - Best Practice](https://docs.datadoghq.com/logs/faq/log-parsing-best-practice/)
- [Datadog Logging without Limits](https://docs.datadoghq.com/logs/logging_without_limits/)
