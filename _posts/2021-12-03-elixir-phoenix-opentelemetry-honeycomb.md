---
layout: post
title: "Elixir Phoenix + OpenTelemetry + Honeycomb"
date: 2021-12-03
---

Want to get instrumenting your Elixir Phoenix application with [OpenTelemetry](https://opentelemetry.io) and send the data to [Honeycomb](https://www.honeycomb.io)?

Look no further! It was a frustrating time trying to adapt the [Erlang OpenTelemetry documentation](https://opentelemetry.io/docs/instrumentation/erlang/) to configure the SDK and send data to Honeycomb. The following should clear up any difficulties you may have in wiring this all together.

First add the following dependencies to your mix.exs file.

{% highlight elixir linenos %}
defp deps do
  {:certifi, "~> 2.8"},
  {:opentelemetry, "~> 1.0.0-rc.3"},
  {:opentelemetry_api, "~> 1.0.0-rc.3.2"},
  {:opentelemetry_ecto, "~> 1.0.0-rc.3"},
  {:opentelemetry_exporter, "~> 1.0.0-rc.3"},
  {:opentelemetry_phoenix, "~> 1.0.0-rc.5"},
end
{% endhighlight %}

Next we need to configure the SDK in your config.exs file. Grab your write key and fill in a dataset name as required.

{% highlight elixir linenos %}
config :opentelemetry, :processors,
  otel_batch_processor: %{
    exporter: {
      :opentelemetry_exporter, %{
        endpoints: [
          {:https, 'api.honeycomb.io', 443, [
            verify: :verify_peer,
            cacertfile: :certifi.cacertfile(),
            depth: 3,
            customize_hostname_check: [
              match_fun: :public_key.pkix_verify_hostname_match_fun(:https)
            ],
          ]},
        ],
        headers: [
          {"x-honeycomb-team", honeycomb_write_key},
          {"x-honeycomb-dataset", honeycomb_dataset}
        ],
        protocol: :grpc,
      }
    }
  }
{% endhighlight %}

Once this is complete you should be good to go. Hit up your Honeycomb dashboard to see your telemetry data in all of its glory!
