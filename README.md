# Solving the problem of logs rejection in OpenSearch/ElasticSearch due to field type conflict when parsing `log` field in fluent-bit

We use the [official fluent-bit helm chart](https://github.com/fluent/helm-charts/tree/main/charts/fluent-bit) and deploy it to Kubernetes.
So, the script itself can be found [here](./values_luaScripts.yaml). And the settings for passing it to fluent-bit are [here](./values_config.yaml).
The settings are optimized for our load and now it specifies a large write-to-file cache, if you don't need that, remove those settings.

So, what does this script do? It takes the `log` field and checks if there is a JSON string there. If there isn't, it doesn't do anything. But if there
is a JSON string, it starts parsing it and serializes all nested objects into a string with a dot. Thus the data type of all keys and values become strings,
except for those strings where the content contains timestamp, and any strings with timestamp OpenSearch/ElasticSearch (OS/ES) assigns the `date` type. This
behavior can be disabled at the `index template` level.

Also note that in our setup, fluent-bit will replace any dots with underscores: `Replace_Dots On`. Also we do not enable the built-in parser: `Merge_Log Off`,
but it is disabled by default and you can remove the setting. The built-in parser perfectly parses the `log` field and its subobjects, however, sometimes the
field types of object values can be different and this is what causes OS/ES to refuse to accept logs.
