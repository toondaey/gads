## Translation from console to cloud

To use the gcloud command from your computer, you must first of all be authenticated using:

```bash
gcloud auth login
```

The command above redirects you to your browser where you complete the login.

If using **cloud shell**, the above is unnecessary.

The next thing is to get the zone where the engine will be deployed (if not known). This can be done using 

```bash
gcloud compute zones list
```

After selecting the zone of choice, we can then go ahead and use it in creating our instance. This can be done either by setting the default zone using:

```bash 
gcloud config set compute/zone [ZONE_NAME]
```

then create the instance using

```bash
gcloud compute instances create [INSTANCE_NAME] \
--machine-type=[MACHINE_TYPE] \
--image-project=[IMAGE_PROJECT] \
--image=[IMAGE] \
--subnet=[SUBNET]
```

This should be done with caution though as subsequent machines will also be created in these zones.

The other method is to use the `--zone=[ZONE_NAME]` flag in the create command above.

The command above only uses some of the flags available for create compute instances. Other flags can be found [here](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create) or by using the command `gcloud compute instances create --help`.

**P.S.**: If your account has multiple projects, please don't forget to set the project you're working on, using `gcloud config set project [PROJECT_ID]` as the commands above may fail otherwise.