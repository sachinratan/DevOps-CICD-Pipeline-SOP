## This guide will provide the useful Kubernetes node troubleshooting related commands.

Debug Pod Approach
You can create a debug pod with network access to the node : 

bash

kubectl debug node/<node-name> -it --image=<debug-image> -- /bin/sh

