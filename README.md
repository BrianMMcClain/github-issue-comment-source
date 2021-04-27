github-issue-comment-source
===

A Knative event source that polls the comments of a GitHub issue, emitting an event for each comment

Arguments
---
- `owner`: The repo owner name (ie. "BrianMMcClain")
- `repo`: The repo name (ie. "github-issue-comment-source")
- `issue`: The issue number in the repo to track (id. "1")
- `interval` How frequent (in seconds) to poll the issue (ie. "90")

If the `--fromBeginning` flag is included, an event will be emitted for each pre-existing comment once the event source is started. If omitted, only new comments will be emitted.

**NOTE:** As these are unauthenticated requests, you'll be faced with GitHub's API rate limit of 60 requests per hour. As such, it's recommended that you set the `interval` argument to be no more frequent than 60 seconds.

Deploy the Event Source
---

After filling in the arguments in the `source.yaml` file, as well as the sink to which events should be sent to, you can deploy the event source with the `kubectl` CLI:

```
kubectl apply -f source.yaml
```

Testing Locally
---

If you'd like to test locally, you can use a basic Knative service which logs POST requests to verify that the events are being emitted properly. You can use the [kn CLI](https://knative.dev/development/client/install-kn/) to deploy the service:

```
kn service create logger --image brianmmcclain/event-logger-service --port 4567
```

Then, in another terminal, follow the logs from the `logger` service:

```
kubectl logs -l serving.knative.dev/service=logger -c user-container -f
```

Finally, deploy the event source:

```
kubectl apply -f source-test.yaml
```
Once deployed, you should see output that looks similar to the following:

```
D, [2021-04-27T16:52:44.143432 #1] DEBUG -- : --- START POST ---
D, [2021-04-27T16:52:44.143486 #1] DEBUG -- : {"message":"Here's a comment","id":827708782,"user":"BrianMMcClain","timestamp":"2021-04-27T15:41:25Z","url":"https://github.com/BrianMMcClain/github-issue-comment-source/issues/1#issuecomment-827708782","owner":"BrianMMcClain","repo":"github-issue-comment-source","issue":"1"}
D, [2021-04-27T16:52:44.143508 #1] DEBUG -- : --- END POST ---
10.244.0.10, 10.244.0.5 - - [27/Apr/2021:16:52:44 +0000] "POST / HTTP/1.1" 200 277 0.0021
127.0.0.1 - - [27/Apr/2021:16:52:44 UTC] "POST / HTTP/1.1" 200 277
- -> /
D, [2021-04-27T16:52:44.153934 #1] DEBUG -- : --- START POST ---
D, [2021-04-27T16:52:44.153978 #1] DEBUG -- : {"message":"And here's another one!","id":827708866,"user":"BrianMMcClain","timestamp":"2021-04-27T15:41:33Z","url":"https://github.com/BrianMMcClain/github-issue-comment-source/issues/1#issuecomment-827708866","owner":"BrianMMcClain","repo":"github-issue-comment-source","issue":"1"}
D, [2021-04-27T16:52:44.154020 #1] DEBUG -- : --- END POST ---
```

To clean up you can delete the event source with:

```
kubectl delete -f source-test.yaml
```

And you can delete the logger service with:

```
kn service delete logger
```
