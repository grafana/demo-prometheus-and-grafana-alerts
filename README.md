# Demo Alerting in Prometheus and Grafana 

Grafana Alerting is built on the Prometheus Alerting model. This demo project showcases the similarities between Prometheus and Grafana alerting systems, covering topics such as:

- Creating alerts in Prometheus
- Recreating the same alerts using Grafana
- Setting up alerts based on Loki logs
- Exploring alerting components like evaluation groups and notification policies
- Creating template notifications
- And more!

This project pairs well with this [Alerting Presentation Template](https://docs.google.com/presentation/d/1XvJnBlNnXUjiS409ABN4NxNkFZoYDmoRKKoJqsvln-g/edit?usp=sharing). Together, they provide an excellent starting point for presenting the Prometheus Alerting model and demonstrating its use in Grafana.

## Run the demo environment

This repository includes a [Docker Compose setup](./docker-compose.yaml) that runs Grafana, Prometheus, Prometheus Alertmanager, Loki, and an SMTP server for testing email notifications.

To run the demo environment:

```bash
docker compose up
```

You can then access:
- Grafana: [http://localhost:3000](http://localhost:3000/)
- Prometheus web UI: [http://localhost:9090](http://localhost:9090/)
- Alertmanager web UI: [http://localhost:9093](http://localhost:9093/)

### Prometheus Alerting plugin

Docker Compose installs and enables the [Prometheus Alerting plugin](https://grafana.com/grafana/plugins/grafana-prometheusalerting-app/) (requires Grafana 13.1 or later). Log in with `admin`/`admin` to use it.

The demo uses vanilla Prometheus and Alertmanager, which have limits in the plugin:
- Rules are read-only, because Prometheus has no ruler API. Edit them in [`prometheus/rules/`](./prometheus/rules/).
- Templates are listed, but their content isn't shown, because Alertmanager only reports template file paths. Edit them in [`alertmanager/templates/`](./alertmanager/templates/). List each new template file in [`alertmanager.yml`](./alertmanager/alertmanager.yml), since a glob appears as a single entry.
- The rules in [`5.routing.yml`](./prometheus/rules/5.routing.yml) always fire, so alerts, routes, and the inhibition rule show up without running the k6 tests.

### Generating test data

This demo uses [Grafana k6](https://grafana.com/docs/k6) to generate test data for Prometheus and Loki.

The [k6 tests in the `testdata` folder](./testdata/) inject Prometheus metrics and Loki logs that you can use to define alert queries and conditions. 

1. Build a k6 binary with the required extensions.

    The scripts import `k6/x/remotewrite` and `k6/x/loki`, which are k6 extensions. Automatic extension resolution doesn't provision them, because the registry flags both with compliance issues.

    Install [xk6](https://github.com/grafana/xk6) and build a local `./k6` binary:

    ```bash
    go install go.k6.io/xk6/cmd/xk6@latest
    xk6 build \
      --with github.com/grafana/xk6-client-prometheus-remote@v0.3.2 \
      --with github.com/grafana/xk6-loki@v1.0.0
    ```

2. Run a k6 test with the following command:

    ```bash
    ./k6 run testdata/<FILE>.js
    ```

You can modify and run the k6 scripts to simulate different alert scenarios.
For details on inserting data into Prometheus or Loki, see the `xk6-client-prometheus-remote` and `xk6-loki` APIs.

### Receive webhook notifications

One of the simplest ways to receive alert notifications is by using a Webhook.  You can use [`webhook.site`](https://webhook.site/) to create Webhook URLs and view the incoming messages.

- For Prometheus alertmanager: 
  
  Set the Webhook URL to the [alertmanager.yml](./alertmanager/alertmanager.yml) configuration file.

- For Grafana:
  
  Create a Webhook contact point and assign it to the notification policy.

### Receive mail notifications

You can also configure notifications to be sent via your Gmail account using an [App Password](https://support.google.com/accounts/answer/185833?hl=en). After creating your App password:

- For Prometheus Alertmanager:

  Replace `your_mail@gmail` with your Gmail address in the [alertmanager.yml](./alertmanager/alertmanager.yml) configuration file.

  Copy `alertmanager/smtp_auth_password.example` to `alertmanager/smtp_auth_password` and set your password.

- For Grafana:

  Copy `environments/smpt.env.example` to `environments/smpt.env` and set the appropriate environment variables values.