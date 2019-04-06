# Popeye as a Sonobuoy plugin

I created this just as an experiment to see how simple it would be to make a [Sonobuoy][sonobuoy] plugin from [popeye][popeye].

The process is pretty simple:

 - the entire `popeye` repo is nested in this repo. Could be vendored code. Could be grabbed in some other way. It doesn't matter, but this was simple to do
 - the app needed to be built into an image, so I created a multi-stage Dockerfile which builds popeye and then puts it into an Ubuntu image. Could be a smaller image, but I wanted one that was easy to work with after the fact if I wanted to exec into it.
 - When calling the image as a plugin I needed to print the output to the `/tmp/results` (I chose a file named "output") and then inform Sonobuoy where that file is by writing the path to `/tmp/results/done`. I did that in a simple script, `run.sh`
 - Build the image with
```
export REGISTRY=schnake
docker build . -t $REGISTRY/sonobuoy-popeye:v0.1
docker push $REGISTRY/sonobuoy-popeye:v0.1
```
 - I had to tell Sonobuoy to run my plugin so I ran:
   - `sonobuoy gen > popeye.yaml`
   - manually edited the file to run my image and run the `./run.sh` command
   - Run it with `kubectl create -f popeye.yaml`

You can check on the status with `sonobuoy status` (I use [watch][watch]; it shouldn't take long for it to complete and the cluster to be queried for other data of interest (one fo the benefits of Sonobuoy).

You can gather all the data and review it via:
```
outfile=$(sonobuoy retrieve)
mkdir results && tar -xf $outfile -C results
```

Then open the results directory with your favorite editor.

[sonobuoy]: https://github.com/heptio/sonobuoy
[watch]: http://osxdaily.com/2010/08/22/install-watch-command-on-os-x/
[popeye]: https://github.com/derailed/popeye