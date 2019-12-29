Decode Gray code pattern tutorial {#tutorial_decode_graycode_pattern}
=============

Goal
----

In this tutorial you will learn how to use the *GrayCodePattern* class to:

-   Decode a previously acquired Gray code pattern.
-   Generate a disparity map.
-   Generate a pointcloud.

在本教程中，您将学习如何使用GrayCodePattern类来：

    解码先前获取的格雷码图案。
    生成视差图。
    生成一个点云。

Code
----

@include structured_light/samples/pointcloud.cpp

Explanation
-----------

First of all the needed parameters must be passed to the program.
The first is the name list of previously acquired pattern images, stored in a .yaml file organized as below:
首先，必须将所需的参数传递给程序。第一个是先前获取的图案图像的名称列表，存储在一个.yaml文件中，其组织如下：

@code{.cpp}
%YAML:1.0
cam1:
   - "/data/pattern_cam1_im1.png"
   - "/data/pattern_cam1_im2.png"
           ..............
   - "/data/pattern_cam1_im42.png"
   - "/data/pattern_cam1_im43.png"
   - "/data/pattern_cam1_im44.png"
cam2:
   - "/data/pattern_cam2_im1.png"
   - "/data/pattern_cam2_im2.png"
           ..............
   - "/data/pattern_cam2_im42.png"
   - "/data/pattern_cam2_im43.png"
   - "/data/pattern_cam2_im44.png"
@endcode

For example, the dataset used for this tutorial has been acquired using a projector with a resolution of 1280x800, so 42 pattern images (from number 1 to 42) + 1 white (number 43) and 1 black (number 44) were captured with both the two cameras.
例如，本教程中使用的数据集是使用分辨率为1280x800的投影仪获取的，因此使用这两种方式分别捕获了42个图案图像（从1到42）+1个白色（43号）和1个黑色（44号）两个相机。

Then the cameras calibration parameters, stored in another .yml file, together with the width and the height of the projector used to project the pattern, and, optionally, the values of white and black tresholds, must be passed to the tutorial program.
然后，必须将存储在另一个.yml文件中的相机校准参数以及用于投影该图案的投影仪的宽度和高度以及可选的白色和黑色阈值的值传递到教程程序。

In this way, *GrayCodePattern* class parameters can be set up with the width and the height of the projector used during the pattern acquisition and a pointer to a GrayCodePattern object can be created:
通过这种方式，可以使用在模式获取期间使用的投影仪的宽度和高度来设置GrayCodePattern类参数，并可以创建指向GrayCodePattern对象的指针：

@code{.cpp}
  structured_light::GrayCodePattern::Params params;
     ....
  params.width = parser.get<int>( 2 );
  params.height = parser.get<int>( 3 );
     ....
  // Set up GraycodePattern with params
  Ptr<structured_light::GrayCodePattern> graycode = structured_light::GrayCodePattern::create( params );
@endcode

If the white and black thresholds are passed as parameters (these thresholds influence the number of decoded pixels), their values can be set, otherwise the algorithm will use the default values.
如果将白色和黑色阈值作为参数传递（这些阈值影响解码像素的数量），则可以设置它们的值，否则算法将使用默认值。
@code{.cpp}
  size_t white_thresh = 0;
  size_t black_thresh = 0;
  if( argc == 7 )
  {
    // If passed, setting the white and black threshold, otherwise using default values
    white_thresh = parser.get<size_t>( 4 );
    black_thresh = parser.get<size_t>( 5 );
    graycode->setWhiteThreshold( white_thresh );
    graycode->setBlackThreshold( black_thresh );
  }
@endcode

At this point, to use the *decode* method of *GrayCodePattern* class, the acquired pattern images must be stored in a vector of vector of Mat.
The external vector has a size of two because two are the cameras: the first vector stores the pattern images captured from the left camera, the second those acquired from the right one. The number of pattern images is obviously the same for both cameras and can be retrieved using the getNumberOfPatternImages() method:
此时，要使用GrayCodePattern类的解码方法，必须将获取的图案图像存储在Mat的矢量中。外部矢量的大小为2，因为有两个是相机：第一个矢量存储从左侧相机捕获的图案图像，第二个矢量存储从右侧相机获取的图案图像。两种相机的图案图像数量明显相同，可以使用getNumberOfPatternImages（）方法进行检索：
@code{.cpp}
  size_t numberOfPatternImages = graycode->getNumberOfPatternImages();
  vector<vector<Mat> > captured_pattern;
  captured_pattern.resize( 2 );
  captured_pattern[0].resize( numberOfPatternImages );
  captured_pattern[1].resize( numberOfPatternImages );

  .....

  for( size_t i = 0; i < numberOfPatternImages; i++ )
  {
    captured_pattern[0][i] = imread( imagelist[i], IMREAD_GRAYSCALE );
    captured_pattern[1][i] = imread( imagelist[i + numberOfPatternImages + 2], IMREAD_GRAYSCALE );
     ......
  }
@endcode

As regards the black and white images, they must be stored in two different vectors of Mat:
关于黑白图像，它们必须存储在Mat的两个不同向量中：
@code{.cpp}
  vector<Mat> blackImages;
  vector<Mat> whiteImages;
  blackImages.resize( 2 );
  whiteImages.resize( 2 );
  // Loading images (all white + all black) needed for shadows computation
  cvtColor( color, whiteImages[0], COLOR_RGB2GRAY );
  whiteImages[1] = imread( imagelist[2 * numberOfPatternImages + 2], IMREAD_GRAYSCALE );
  blackImages[0] = imread( imagelist[numberOfPatternImages + 1], IMREAD_GRAYSCALE );
  blackImages[1] = imread( imagelist[2 * numberOfPatternImages + 2 + 1], IMREAD_GRAYSCALE );
@endcode

It is important to underline that all the images, the pattern ones, black and white, must be loaded as grayscale images and rectified before being passed to decode method:
重要的是要强调所有图像（黑白图像）必须作为灰度图像加载并进行校正，然后才能传递给解码方法：
@code{.cpp}
  // Stereo rectify
  cout << "Rectifying images..." << endl;
  Mat R1, R2, P1, P2, Q;
  Rect validRoi[2];
  stereoRectify( cam1intrinsics, cam1distCoeffs, cam2intrinsics, cam2distCoeffs, imagesSize, R, T, R1, R2, P1, P2, Q, 0,
                -1, imagesSize, &validRoi[0], &validRoi[1] );
  Mat map1x, map1y, map2x, map2y;
  initUndistortRectifyMap( cam1intrinsics, cam1distCoeffs, R1, P1, imagesSize, CV_32FC1, map1x, map1y );
  initUndistortRectifyMap( cam2intrinsics, cam2distCoeffs, R2, P2, imagesSize, CV_32FC1, map2x, map2y );
         ........
  for( size_t i = 0; i < numberOfPatternImages; i++ )
  {
         ........
    remap( captured_pattern[1][i], captured_pattern[1][i], map1x, map1y, INTER_NEAREST, BORDER_CONSTANT, Scalar() );
    remap( captured_pattern[0][i], captured_pattern[0][i], map2x, map2y, INTER_NEAREST, BORDER_CONSTANT, Scalar() );
  }
         ........
  remap( color, color, map2x, map2y, INTER_NEAREST, BORDER_CONSTANT, Scalar() );
  remap( whiteImages[0], whiteImages[0], map2x, map2y, INTER_NEAREST, BORDER_CONSTANT, Scalar() );
  remap( whiteImages[1], whiteImages[1], map1x, map1y, INTER_NEAREST, BORDER_CONSTANT, Scalar() );
  remap( blackImages[0], blackImages[0], map2x, map2y, INTER_NEAREST, BORDER_CONSTANT, Scalar() );
  remap( blackImages[1], blackImages[1], map1x, map1y, INTER_NEAREST, BORDER_CONSTANT, Scalar() );
@endcode


In this way the *decode* method can be called to decode the pattern and to generate the corresponding disparity map, computed on the first camera (left):

@code{.cpp}
  Mat disparityMap;
  bool decoded = graycode->decode(captured_pattern, disparityMap, blackImages, whiteImages,
                                  structured_light::DECODE_3D_UNDERWORLD);
@endcode

To better visualize the result, a colormap is applied to the computed disparity:
@code{.cpp}
    double min;
    double max;
    minMaxIdx(disparityMap, &min, &max);
    Mat cm_disp, scaledDisparityMap;
    cout << "disp min " << min << endl << "disp max " << max << endl;
    convertScaleAbs( disparityMap, scaledDisparityMap, 255 / ( max - min ) );
    applyColorMap( scaledDisparityMap, cm_disp, COLORMAP_JET );
    // Show the result
    resize( cm_disp, cm_disp, Size( 640, 480 ) );
    imshow( "cm disparity m", cm_disp )
@endcode

![](pics/cm_disparity.png)

At this point the point cloud can be generated using the reprojectImageTo3D method, taking care to convert the computed disparity in a CV_32FC1 Mat (decode method computes a CV_64FC1 disparity map):
此时，可以使用reprojectImageTo3D方法生成点云，注意转换CV_32FC1 Mat中的计算出的视差（解码方法会计算CV_64FC1视差图）：

@code{.cpp}
  Mat pointcloud;
  disparityMap.convertTo( disparityMap, CV_32FC1 );
  reprojectImageTo3D( disparityMap, pointcloud, Q, true, -1 );
@endcode

Then a mask to remove the unwanted background is computed:
然后计算一个去除不需要的背景的遮罩：
@code{.cpp}
  Mat dst, thresholded_disp;
  threshold( scaledDisparityMap, thresholded_disp, 0, 255, THRESH_OTSU + THRESH_BINARY );
  resize( thresholded_disp, dst, Size( 640, 480 ) );
  imshow( "threshold disp otsu", dst );
@endcode
![](pics/threshold_disp.png)

The white image of cam1 was previously loaded also as a color image, in order to map the color of the object on its reconstructed pointcloud:
cam1的白色图像先前也已作为彩色图像加载，以便将对象的颜色映射到其重构的点云上：
@code{.cpp}
  Mat color = imread( imagelist[numberOfPatternImages], IMREAD_COLOR );
@endcode

The background renoval mask is thus applied to the point cloud and to the color image:
因此，背景修复蒙版将应用于点云和彩色图像：

@code{.cpp}
  Mat pointcloud_tresh, color_tresh;
  pointcloud.copyTo(pointcloud_tresh, thresholded_disp);
  color.copyTo(color_tresh, thresholded_disp);
@endcode

Finally the computed point cloud of the scanned object can be visualized on viz:
最后，可以在可视化上可视化扫描对象的计算点云：
@code{.cpp}
  viz::Viz3d myWindow( "Point cloud with color");
  myWindow.setBackgroundMeshLab();
  myWindow.showWidget( "coosys", viz::WCoordinateSystem());
  myWindow.showWidget( "pointcloud", viz::WCloud( pointcloud_tresh, color_tresh ) );
  myWindow.showWidget( "text2d", viz::WText( "Point cloud", Point(20, 20), 20, viz::Color::green() ) );
  myWindow.spin();
@endcode

![](pics/plane_viz.png)
