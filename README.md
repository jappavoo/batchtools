# How to run a batch job on the run container

`run "oc rsync csw-dev-0:$(pwd)/ .; ./hello > out.$$; oc rsync . csw-dev-0:$(pwd)/"`
