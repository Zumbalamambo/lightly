.. _rst-docker-known-issues-faq:

Known Issues and FAQ
===================================

Docker is slow when working with long videos
---------------------------------------------------

We are working on this issue internally. For now we suggest to split the large
videos into chunks. You can do this using ffmpeg and without losing quality.
The following code just breaks up the video in a way that no re-encoding is needed.

.. code-block:: console

    ffmpeg -i input.mp4 -c copy -map 0 -segment_time 01:00:00 -f segment -reset_timestamps 1 output%03d.mp4

What exactly happens here?

- `input.mp4`, this is your input video
- `-c copy -map 0`, this makes sure we just copy and don't re-encode the video
- `-segment_time 01:00:00 -f segment`, defines that we want chunks of 1h each
- `-reset_timestamps 1`, makes sure we reset the timestamps (each video starts from 0)
- `output%03d.mp4`, name of the output vidoes (output001.mp4, output002.mp4, ...)


Shared Memory Error when running Lightly Docker
-----------------------------------------------

The following error message appears when the docker runtime has not enough
shared memory. By default Docker uses 64 MBytes. However, when using multiple 
workers for data fetching `lightly.loader.num_workers` there might be not enough.

.. code-block:: console

    ERROR: Unexpected bus error encountered in worker. This might be caused by insufficient shared memory (shm).                                                                                                
    Traceback (most recent call last):                                                                                                                                                                          
    File "/opt/conda/envs/env/lib/python3.7/multiprocessing/queues.py", line 236, in _feed                                                                                                                    
        obj = _ForkingPickler.dumps(obj)                                                                                                                                                                        
    File "/opt/conda/envs/env/lib/python3.7/multiprocessing/reduction.py", line 51, in dumps                                                                                                                  
        cls(buf, protocol).dump(obj)                                                                                                                                                                            
    File "/opt/conda/envs/env/lib/python3.7/site-packages/torch/multiprocessing/reductions.py", line 321, in reduce_storage                                                                                   
        fd, size = storage._share_fd_()                                                                                                                                                                         
    RuntimeError: unable to write to file </torch_31_1030151126> 

To solve this problem we need to increase the shared memory for the docker runtime.

You can change the shared memory to 512 MBytes by adding `--shm-size="512m"` to 
the docker run command:

.. code-block:: console

    # example of docker run with setting shared memory to 512 MBytes
    docker run --shm-size="512m" --gpus all