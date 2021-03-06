# Security Barrier Camera Demo

This demo showcases Vehicle and License Plate Detection network followed by the Vehicle Attributes Recognition and License Plate Recognition networks applied on top
of the detection results. The corresponding topologies are shipped with the product:
* `vehicle-license-plate-detection-barrier-0106`, which is a primary detection network to find the vehicles and license plates
* `vehicle-attributes-recognition-barrier-0039`, which is executed on top of the results from the first network and
reports general vehicle attributes, for example, vehicle type (car/van/bus/track) and color
* `license-plate-recognition-barrier-0001`, which is executed on top of the results from the first network
and reports a string per recognized license plate

For more details on the topologies, please refer to the descriptions in the `open_model_zoo/intel_models` folder.

Other demo objectives are:
* Video/Camera as inputs, via OpenCV\*
* Example of simple asynchronous networks pipelining: Vehicle Attributes and License Plate Recognition networks are executed on top of the Vehicle Detection results
* Visualization of Vehicle Attributes and License Plate information for each detected object


### How It Works

On the start-up, the application reads command line parameters and loads the specified networks. The Vehicle and License Plate
Detection network is required, the other two are optional.

Upon getting a frame from the OpenCV VideoCapture, the applications performs inference of Vehicles and License Plate Detection network, then performs
another two inferences of Vehicle Attributes and License Plate Recognition networks if they were specified in command line, and displays the results.

## Running

Running the application with the <code>-h</code> option yields the following usage message:
```sh
./security_barrier_camera_demo -h
InferenceEngine:
    API version ............ <version>
    Build .................. <number>

interactive_vehicle_detection [OPTION]
Options:

    -h                         Print a usage message.
    -i "<path1>" "<path2>"     Required. Path to video or image files. Default value is "cam" to work with cameras.
    -m "<path>"                Required. Path to the Vehicle and License Plate Detection model .xml file.
    -m_va "<path>"             Optional. Path to the Vehicle Attributes model .xml file.
    -m_lpr "<path>"            Optional. Path to the License Plate Recognition model .xml file.
      -l "<absolute_path>"     Optional. For CPU custom layers, if any. Absolute path to a shared library with the kernels implementation.
          Or
      -c "<absolute_path>"     Optional. For GPU custom kernels, if any. Absolute path to an .xml file with the kernels description.
    -d "<device>"              Optional. Specify the target device for Vehicle Detection (CPU, GPU, FPGA, MYRIAD, or HETERO).
    -d_va "<device>"           Optional. Specify the target device for Vehicle Attributes (CPU, GPU, FPGA, MYRIAD, or HETERO).
    -d_lpr "<device>"          Optional. Specify the target device for License Plate Recognition (CPU, GPU, FPGA, MYRIAD, or HETERO).
    -pc                        Optional. Enable per-layer performance statistics.
    -r                         Optional. Output inference results as raw values.
    -t                         Optional. Probability threshold for vehicle and license plate detections.
    -no_show                   Optional. Do not show processed video.
    -auto_resize               Optional. Enable resizable input with support of ROI crop and auto resize.
    -nireq                     Optional. Number of infer request for pipelined mode (default value is 1)
    -nc                        Optional. Number of processed cameras (default value is 1) if the input (-i) is specified as camera.
```

Running the application with an empty list of options yields the usage message given above and an error message.

To run the demo, you can use public models or a set of pre-trained and optimized models:

* `open_model_zoo/intel_models/vehicle-license-plate-detection-barrier-0106`
* `open_model_zoo/intel_models/vehicle-attributes-recognition-barrier-0039`
* `open_model_zoo/intel_models/license-plate-recognition-barrier-0001`

For example, to do inference on a GPU with the pre-trained models, run the following command:

```sh
./security_barrier_camera_demo -i <path_to_video>/inputVideo.mp4 -m vehicle-license-plate-detection-barrier-0106.xml -m_va vehicle-attributes-recognition-barrier-0039.xml -m_lpr license-plate-recognition-barrier-0001.xml -d GPU
```

To do inference for two video inputs using two asynchronous infer request on FPGA with the pre-trained models, run the following command:
```sh
./security_barrier_camera_demo -i <path_to_video>/inputVideo_0.mp4 <path_to_video>/inputVideo_1.mp4 -m vehicle-license-plate-detection-barrier-0106.xml -m_va vehicle-attributes-recognition-barrier-0039.xml -m_lpr license-plate-recognition-barrier-0001.xml -d HETERO:FPGA,CPU -d_va HETERO:FPGA,CPU -d_lpr HETERO:FPGA,CPU -nireq 2
```

> **NOTE**: Public models must be first converted to the Inference Engine format (`.xml` + `.bin`) using the [Model Optimizer tool](https://software.intel.com/en-us/articles/OpenVINO-ModelOptimizer).


### Optimization Hints for Heterogeneous Scenarios with FPGA

* `OMP_NUM_THREADS`: Specifies number of threads to use. For heterogeneous scenarios with FPGA, when several inference requests are used asynchronously, limiting the number of CPU threads with `OMP_NUM_THREADS` allows to avoid competing for resources between threads. For the Security Barrier Camera Demo, recommended value is `OMP_NUM_THREADS=1`.
* `KMP_BLOCKTIME`: Sets the time, in milliseconds, that a thread should wait, after completing the execution of a parallel region, before sleeping. The default value is 200ms, which is not optimal for the demo. Recommended value is `KMP_BLOCKTIME=1`.


### Demo Output

The demo uses OpenCV to display the resulting frame with detections rendered as bounding boxes and text.

## See Also
* [Using Inference Engine Demos](../Readme.md)
