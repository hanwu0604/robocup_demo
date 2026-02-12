**A supplement to the tutorial**

**1. Configuration environment**

Need to configure it on the system environment, do conda deactivate
until there is no (Env\_name) beyond the terminal line.

**2.CUDA version mismatch**

If the CUDA version in your system environment is not 12.2, then you
must set the CUDA path manually using environment variables(Minimum
Requirement:11.8):

export CUDAToolkit\_ROOT=/usr/local/cuda-12.2

export CUDA\_HOME=\$CUDAToolkit\_ROOT

export PATH=\$CUDA\_HOME/bin:\$PATH

export LD\_LIBRARY\_PATH=\$CUDA\_HOME/lib64:\$LD\_LIBRARY\_PATH

**3.TensorRT verison:**\
Why do not use TensorRT8.6?

Because under **RTX 5060 + CUDA 12.x**, TensorRT 8.6 is no longer
technically supported.

Error Code 2: Internal Error

Assertion major \>= 0 && major \< 10 failed

The codebase currently supports only TensorRT **8.6** and **10.3**.

If a different TensorRT version is installed, the RoboCup demo will fail
to compile.

In that case, you need to modify the corresponding elif conditions in
the following file to match your installed TensorRT version.

～/Workspace/booster/robocup\_demo

src/vision/include/booster\_vision/model/trt/impl.h

src/vision/src/model/trt/impl.cpp

src/vision/src/model/trt/yolov8\_det.cpp

Find:

\#elif (NV\_TENSORRT\_MAJOR == 10) && (NV\_TENSORRT\_MINOR = 3)

Change it to

\#elif (NV\_TENSORRT\_MAJOR == 10) && (NV\_TENSORRT\_MINOR \>=3)

**4. If there is an error related to NvInferVersion.h**

Error

/home/han/Workspace/booster/robocup\_demo/src/vision/src/model/trt/postprocess.cpp:1:10:
fatal error: NvInferVersion.h: No such file or directory

1 \| \#include \<NvInferVersion.h\>

**The most straightforward solution is to explicitly force the project
to use the TensorRT 10.8 include and library paths.**

Open the following file ：\
src/vision/src/model/trt/CMakeLists.txt

Add the following at the top (adjust the path to match your TensorRT
installation):

set(TENSORRT\_ROOT /usr/local/TensorRT-10.8.0.43)

include\_directories(\${TENSORRT\_ROOT}/include)

link\_directories(\${TENSORRT\_ROOT}/lib)

Then update target\_link\_libraries to make the TensorRT linkage more
explicit (to ensure it resolves to TensorRT 10.8):

target\_link\_libraries(yolov8\_trt PRIVATE nvinfer nvinfer\_plugin
nvonnxparser cudart PUBLIC \${OpenCV\_LIBS})

link\_directories(\${TENSORRT\_ROOT}/lib)

the linker will prioritize resolving nvinfer from the TensorRT 10.8
library directory.

Otherwise, the system may accidentally pick up another version (e.g.,
libnvinfer.so.8) from a different system path, which can cause build or
runtime conflicts.

**5.Isaacsim version**

It is recommended to install Isaac Sim 4.x for this project.

(https://docs.isaacsim.omniverse.nvidia.com/4.5.0/installation/download.html）\
The RoboCup demo relies heavily on NVIDIA Omniverse Kit extensions
(omni.\*). These extensions were originally developed for Isaac Sim 4.2
and are tightly coupled with that version of the underlying Kit
framework.

Running the demo on Isaac Sim 5.x often leads to compatibility issues,
as many APIs and extension interfaces have changed. As a result, the
demo may fail to load or certain components may not function properly.

**6.Build a TensorRT engine from the model.**

If the TensorRT version you are using is not **8.6**, the script
provided in the instructions will **not** be able to generate the
engine. Also, this repository only includes a prebuilt engine file
(bestOrin.engine). Therefore, I recommend generating the TensorRT engine
yourself.

### **A) Use the official TensorRT trtexec tool to generate the engine**

This is the most stable, fastest, and most compatible method with your
GPU and TensorRT version.

You already have model.wts, but trtexec usually does not accept .wts
files directly. It expects an **ONNX** model (or other supported
formats). Therefore, you need to first export det.pt to ONNX, then use
trtexec to convert it into a TensorRT engine.

### **A1) Export ONNX (inside the conda environment)**

Set up environment should include PyTorch, Ultralytics, and related
dependencies.

cd \~/Workspace/booster/robocup\_demo

\# Verify det.pt exists (your script uses scripts/vision/model/det.pt)

ls scripts/vision/model/det.pt

If this is a YOLOv8 .pt model (Ultralytics), export it like this:

python - \<\<\'PY\'

from ultralytics import YOLO

m = YOLO(\"scripts/vision/model/det.pt\")

m.export(format=\"onnx\", opset=13, simplify=True) \# generates det.onnx

PY

After exporting, verify:

ls -lah \*.onnx scripts/vision/model/\*.onnx

If you see No module named ultralytics, install dependencies inside the
conda environment:

pip install ultralytics onnx onnxsim

### **A2) Generate the TensorRT engine using trtexec (on RTX 5060)**

First, add trtexec to your PATH:

export PATH=/usr/local/TensorRT-10.8.0.43/bin:\$PATH

trtexec \--version

First, run the following commands in the **root directory of the
repository**:

cd \~/Workspace/booster/robocup\_demo

export PATH=/usr/local/TensorRT-10.8.0.43/bin:\$PATH

trtexec \\

\--onnx=scripts/vision/model/det.onnx \\

\--saveEngine=scripts/vision/exported\_model/model.engine \\

\--fp16 \\

\--memPoolSize=workspace:4096 \\

\--skipInference

### **Explanation of the parameters:**

-   **\--saveEngine=\...\
    > ** Outputs the generated TensorRT engine file.

-   **\--fp16\
    > ** Enables FP16 precision (most RTX GPUs support this).

-   **\--memPoolSize=workspace:4096\
    > ** Allocates sufficient workspace memory (4GB) for the TensorRT
    > builder.

-   **\--skipInference\
    > ** Only builds the engine without running random inference
    > afterward (faster and cleaner).

Once the process is completed successfully, you will find model.engine
in ./scripts/vision/exported\_model folder.

ls ./scripts/vision/exported\_model

Copy model.engine to vision folder.

cp ./scripts/vision/exported\_model/model.engine ./src/vision/model/

**7.Command error on "Start the simulation environment"**

![](media/image1.png){width="5.109375546806649in"
height="4.811519028871391in"}

cd \~/Workspace/tools

./isaac\_package\_0.0.6.run /home/han/isaacsim4.2/python.sh

(recommend pass your isaac path)

cd \~/Workspace/tools ./booster-runner-full-0.0.11.run

note:**Use git checkout with extreme caution.**

It may cause inconsistencies between the repository contents and your
local modifications, which can lead to build or runtime errors. In most
cases, it is not necessary to run git checkout to proceed.
