SHELL := /bin/bash

prow-namespace:
	kubectl apply -f cluster/prow_namespace.yaml

test-pods-namespace:
	kubectl apply -f cluster/test-pods_namespace.yaml

clean:
	rm -f cluster/github-token.yaml
	rm -f cluster/hmac-token.yaml