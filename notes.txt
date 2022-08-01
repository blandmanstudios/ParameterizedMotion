# Notes to get things to run once its all installed

## Commands and things that are working

If you're getting an error when trying to run `./render` because a shared object
library file is not found but it is installed you may need to run the following
command. I think after a reboot this stopped being necessary
```
export LD_LIBRARY_PATH=/usr/local/lib
```


You can render a motion file and watch an example skeleton jump
```
cd build/render
./render --bvh=../../data/motion/jump_mxm.bvh
```
Render one of the pretrained motion files. Pretrained means the model has
learned to do the unmodified motion of the input file
```
./render --bvh=jump_mxm.bvh --ppo=jump_mxm/network-0
```
Render one of the parameterized motion files. Parameterized means it has been
pretrained to do the input motion, but then it has been trained to do a family
of motions. The ones in this example can be customized with gravity and energy

These are pretty fun to play with. If you turn gravity half way up and energy
all the way up there is a funny effect where he jumps so high that he lands hard
and his back breaks when he lands. Also if you turn gravity to 0 and energy half
way up to get some really high jumps that look like moon gravity

Note: you need to click `set(param network)` for your parameters to affect the
simulation/animation. I'm not sure what `set(elite set)` does
```
./render --bvh=jump_mxm.bvh --ppo=jump_mxm_parameterized/network-0 --parametric
```

To start pretraining and training models you need to load the python library
```
source ../../py_env/bin/activate
```

I allowed one of the backflip motions to pretrain for 24 hours. Instead of
setting a number of iterations, I just closed it with a keyboard interrupt when
complete
```
python3 ppo.py --ref=backflip_mxm.bvh --test_name=backflip_mxm_test1 --nslave=4
```
I saved the results of this run by copying `network/output/backflip_mxm_test1`
to `network/output/backflip_mxm_test1_24_hour_run_1259Iteration`

I believe you can use the following command to resume pretraining from the point
where it left off. It will generate fresh new graphs tho
```
python3 ppo.py --ref=backflip_mxm.bvh --test_name=backflip_mxm_test1 --nslave=4 --pretrain=network-0
```

Then I ran the parameterized learning. I let this one run for 140 minutes and
the performance isn't great, as you'll see with the commands later. From the
graphs it looks like the `jump_mxm_parameterized` model ran for like 3000
minutes. So I would guess its expected that by parameterized model doesnt
perform great
```
python3 ppo.py --ref=backflip_mxm.bvh --test_name=backflip_mxm_test1 --nslave=4 --pretrain=network-0 --parametric
```
I saved the results to another directory so they wont be changed by copying
`network/output/backflip_mxm_test1` to
`network/output/backflip_mxm_test1_24_hour_run_1259Iterations_parameterized_for140_minutes`


View the jump and backflip reference files
```
./render --bvh=jump_mxm.bvh
./render --bvh=backflip_mxm.bvh
```


View the pretrained network
The `jump_mxm` one works, but my `backflip_mxm` one flops onto its face. Perhaps
it didn't really learn the motion. This is a pretty complex motion tho,
especially to be able to land
```
./render --bvh=jump_mxm.bvh --ppo=jump_mxm/network-0
./render --bvh=backflip_mxm.bvh --ppo=backflip_mxm_test1_24_hour_run_1259Iterations/network-0
```

View and tweak the parameters for the trained models. I talked above about
tweaking the `jump_mxm` animation. The backflip one is pretty unsuccessful but
I can turn up the energy and make him to a double flip that doesnt land. Pretty
cool stuff. Even though the learning wasnt successful it was pretty entertaining
to "teach" a motion
```
./render --bvh=jump_mxm.bvh --ppo=jump_mxm_parameterized/network-0 --parametric
./render --bvh=backflip_mxm.bvh --ppo=backflip_mxm_test1_24_hour_run_1259Iterations_parameterized_for140_minutes/network-0 --parametric
```


## Errors error codes and fixes

It was failing to load this shared object cuda file like below: libcudart.so.11.```
2022-07-30 16:43:55.374119: W tensorflow/stream_executor/platform/default/dso_loader.cc:60] Could not load dynamic library 'libcudart.so.11.0'; dlerror: libcudart.so.11.0: cannot open shared object file: No such file or directory
```
So I installed CUDA Toolkit 11.7 from instructions here:
https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=18.04&target_type=deb_network

Then it was erroring that I dont have a CUDA device (cuz i was using intel
integrated graphics)
```
2022-07-30 18:07:33.939900: I tensorflow/stream_executor/platform/default/dso_loader.cc:49] Successfully opened dynamic library libcudart.so.11.0
2022-07-30 18:07:44.665180: I tensorflow/compiler/jit/xla_cpu_device.cc:41] Not creating XLA devices, tf_xla_enable_xla_devices not set
2022-07-30 18:07:44.734105: I tensorflow/stream_executor/platform/default/dso_loader.cc:49] Successfully opened dynamic library libcuda.so.1
2022-07-30 18:07:44.946008: E tensorflow/stream_executor/cuda/cuda_driver.cc:328] failed call to cuInit: CUDA_ERROR_NO_DEVICE: no CUDA-capable device is detected
```
So i installed an nvidia GPU and nvidia drivers and i got it working using the commands above

