[English](README.md) | 简体中文

------------------------------------------------------------------------------------------

<p align="left">
    <a href="./LICENSE"><img src="https://img.shields.io/badge/license-Apache%202-dfd.svg"></a>
    <a href="https://github.com/PaddlePaddle/PaddleOCR/releases"><img src="https://img.shields.io/github/v/release/PaddlePaddle/PaddleOCR?color=ffa"></a>
    <a href=""><img src="https://img.shields.io/badge/python-3.7+-aff.svg"></a>
    <a href=""><img src="https://img.shields.io/badge/os-linux%2C%20win%2C%20mac-pink.svg"></a>
    <a href=""><img src="https://img.shields.io/pypi/format/PaddleOCR?color=c77"></a>
    <a href="https://github.com/PaddlePaddle/PaddleOCR/graphs/contributors"><img src="https://img.shields.io/github/contributors/PaddlePaddle/PaddleOCR?color=9ea"></a>
    <a href="https://pypi.org/project/PaddleOCR/"><img src="https://img.shields.io/pypi/dm/PaddleOCR?color=9cf"></a>
    <a href="https://github.com/PaddlePaddle/PaddleOCR/stargazers"><img src="https://img.shields.io/github/stars/PaddlePaddle/PaddleOCR?color=ccf"></a>
</p>

## 简介
conda create --name PaddleOCR python=3.7

conda activate PaddleOCR

conda install cmake



编译OPENCV:

wget https://github.com/opencv/opencv/archive/3.4.7.tar.gz
tar -xf 3.4.7.tar.gz


rm -rf build

mkdir build

cd build

cmake .. -DCMAKE_INSTALL_PREFIX=/data1/OCR/opencv-3.4.7/opencv3 -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DWITH_IPP=OFF -DBUILD_IPP_IW=OFF -DWITH_LAPACK=OFF  -DWITH_EIGEN=OFF -DCMAKE_INSTALL_LIBDIR=lib64 -DWITH_ZLIB=ON  -DBUILD_ZLIB=ON -DWITH_JPEG=ON -DBUILD_JPEG=ON -DWITH_PNG=ON -DBUILD_PNG=ON -DWITH_TIFF=ON -DBUILD_TIFF=ON

make -j

make install


编译Paddle:

git clone https://github.com.cnpmjs.org/PaddlePaddle/Paddle.git

git checkout release/2.1



rm -rf build

mkdir build

cd build

ulimit -n 2048，不然编译会报错，说文件过多

cmake -DFLUID_INFERENCE_INSTALL_DIR=/data1/OCR/Paddle  -DCMAKE_BUILD_TYPE=Release -DWITH_PYTHON=OFF -DWITH_MKL=OFF -DWITH_GPU=OFF -DON_INFER=ON  -DWITH_NCCL=OFF .. (https://www.paddlepaddle.org.cn/documentation/docs/zh/2.0/guides/05_inference_deployment/inference/build_and_install_lib_cn.html#congyuanmabianyi)

编译Paddle会下载一些外部库(git clone)，有可能速度比较慢，或者需要多试几次

会编译时间比较久，主要Paddle依赖的库会比较多

编译后的库和头文件，以及第三方库位于paddle_inference_install_dir文件夹里面



转换模型：

git clone https://github.com/PaddlePaddle/PaddleOCR



python3 -m pip install paddlepaddle==2.0.0 -i https://mirror.baidu.com/pypi/simple
cd PaddleOCR
pip3 install -r requirements.txt


检测模型：



wget -P ./ch_lite/ https://paddleocr.bj.bcebos.com/dygraph_v2.0/ch/ch_ppocr_mobile_v2.0_det_train.tar && tar xf ./ch_lite/ch_ppocr_mobile_v2.0_det_train.tar -C ./ch_lite/

python3 tools/export_model.py -c configs/det/ch_ppocr_v2.0/ch_det_mv3_db_v2.0.yml -o Global.pretrained_model=./ch_lite/ch_ppocr_mobile_v2.0_det_train/best_accuracy Global.save_inference_dir=./inference/det_db/


识别模型：

wget -P ./ch_lite/ https://paddleocr.bj.bcebos.com/dygraph_v2.0/ch/ch_ppocr_mobile_v2.0_rec_train.tar && tar xf ./ch_lite/ch_ppocr_mobile_v2.0_rec_train.tar -C ./ch_lite/

python3 tools/export_model.py -c configs/rec/ch_ppocr_v2.0/rec_chinese_lite_train_v2.0.yml -o Global.pretrained_model=./ch_lite/ch_ppocr_mobile_v2.0_rec_train/best_accuracy  Global.save_inference_dir=./inference/rec_crnn/


方向分类模型：



wget -P ./ch_lite/ https://paddleocr.bj.bcebos.com/dygraph_v2.0/ch/ch_ppocr_mobile_v2.0_cls_train.tar && tar xf ./ch_lite/ch_ppocr_mobile_v2.0_cls_train.tar -C ./ch_lite/

python3 tools/export_model.py -c configs/cls/cls_mv3.yml -o Global.pretrained_model=./ch_lite/ch_ppocr_mobile_v2.0_cls_train/best_accuracy  Global.save_inference_dir=./inference/cls/


编译PaddleOCR：



cd PaddleOCR/deploy/cpp_infer

sh tools/build.sh
修改build.sh为：

OPENCV_DIR=/data1/OCR/opencv-3.4.7/opencv3
LIB_DIR=/data1/OCR/Paddle/build/paddle_inference_install_dir

BUILD_DIR=build
rm -rf ${BUILD_DIR}
mkdir ${BUILD_DIR}
cd ${BUILD_DIR}
cmake .. \
-DPADDLE_LIB=${LIB_DIR} \
-DWITH_MKL=OFF \
-DWITH_GPU=OFF \
-DWITH_STATIC_LIB=OFF \
-DUSE_TENSORRT=OFF \
-DOPENCV_DIR=${OPENCV_DIR} \

make -j



编译完成之后，会在build文件夹下生成一个名为ocr_system的可执行文件



sh tools/run.sh
修改tools里面config.txt里面的模型路径，就可以得到识别结果
