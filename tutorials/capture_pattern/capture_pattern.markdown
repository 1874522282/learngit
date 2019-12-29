Capture Gray code pattern tutorial {#tutorial_capture_graycode_pattern}
=============

Goal
----

In this tutorial you will learn how to use the *GrayCodePattern* class to:

-   Generate a Gray code pattern.
-   Project the Gray code pattern.
-   Capture the projected Gray code pattern.
在本教程中，您将学习如何使用* GrayCodePattern *类来：

-生成格雷码模式。
-投影格雷码图案。
-捕获投影的格雷码图案。

It is important to underline that *GrayCodePattern* class actually implements the 3DUNDERWORLD algorithm described in @cite UNDERWORLD , which is based on a stereo approach: we need to capture the projected pattern at the same time from two different views if we want to reconstruct the 3D model of the scanned object. Thus, an acquisition set consists of the images captured by each camera for each image in the pattern sequence.
需要强调的是，GrayCodePattern类实际上实现了[101]中描述的3DUNDERWORLD算法，该算法基于立体方法：如果要重构3D模型，我们需要同时从两个不同的视图捕获投影的图案。扫描的对象。因此，获取集由每个摄像机针对模式序列中的每个图像捕获的图像组成。
Code
----

@include structured_light/samples/cap_pattern.cpp

Explanation
-----------
First of all the pattern images to project must be generated. Since the number of images is a function of the projector's resolution, *GrayCodePattern* class parameters must be set with our projector's width and height. In this way the *generate* method can be called: it fills a vector of Mat with the computed pattern images:
首先必须生成要投影的图案图像。由于图像数量是投影机分辨率的函数，因此必须使用投影机的宽度和高度设置* GrayCodePattern *类参数。这样，可以调用* generate *方法：它用计算出的图案图像填充Mat的向量：
@code{.cpp}
  structured_light::GrayCodePattern::Params params;
    ....
  params.width = parser.get<int>( 1 );
  params.height = parser.get<int>( 2 );
    ....
  // Set up GraycodePattern with params
  Ptr<structured_light::GrayCodePattern> graycode = structured_light::GrayCodePattern::create( params );
  // Storage for pattern
  vector<Mat> pattern;
  graycode->generate( pattern );
@endcode

For example, using the default projector resolution (1024 x 768), 40 images have to be projected: 20 for regular color pattern (10 images for the columns sequence and 10 for the rows one) and 20 for the color-inverted pattern, where the inverted pattern images are images with the same structure as the original but with inverted colors. This provides an effective method for easily determining the intensity value of each pixel when it is lit (highest value) and when it is not lit (lowest value) during the decoding step.
例如，使用默认的投影仪分辨率（1024 x 768），必须投影40张图像：常规颜色图案20张（列序列10张图像，第一行10张图像）和彩色反转图案20张，其中反转图案图像是与原始图像具有相同结构但具有反转颜色的图像。这提供了一种有效的方法，用于容易地确定每个像素在解码步骤中何时点亮（最高值）和何时不点亮（最低值）的强度值。
Subsequently, to identify shadow regions, the regions of two images where the pixels are not lit by projector's light and thus where there is not code information, the 3DUNDERWORLD algorithm computes a shadow mask for the two cameras views, starting from a white and a black images captured by each camera. So two additional images need to be projected and captured with both cameras:
随后，为了识别阴影区域，即两个图像的区域，其中像素没有被投影机的光线照亮，因此没有代码信息，因此3DUNDERWORLD算法从白色和黑色开始为两个摄像机视图计算阴影蒙版每个摄像机捕获的图像。因此，需要用两个摄像机投影并捕获另外两个图像：
@code{.cpp}
  // Generate the all-white and all-black images needed for shadows mask computation
  Mat white;
  Mat black;
  graycode->getImagesForShadowMasks( black, white );
  pattern.push_back( white );
  pattern.push_back( black );
@endcode

Thus, the final projection sequence is projected as follows: first the column and its inverted sequence, then the row and its inverted sequence and finally the white and black images.
因此，最终的投影序列的投影方式如下：首先是列及其反转序列，然后是行及其反转序列，最后是白色和黑色图像。

Once the pattern images have been generated, they must be projected using the full screen option: the images must fill all the projection area, otherwise the projector full resolution is not exploited, a condition on which is based 3DUNDERWORLD implementation.
生成图案图像后，必须使用全屏选项进行投影：图像必须填满所有投影区域，否则将无法使用投影机的全分辨率，这是基于3DUNDERWORLD实现的条件。

@code{.cpp}
  // Setting pattern window on second monitor (the projector's one)
  namedWindow( "Pattern Window", WINDOW_NORMAL );
  resizeWindow( "Pattern Window", params.width, params.height );
  moveWindow( "Pattern Window", params.width + 316, -20 );
  setWindowProperty( "Pattern Window", WND_PROP_FULLSCREEN, WINDOW_FULLSCREEN );
@endcode

At this point the images can be captured with our digital cameras, using libgphoto2 library, recently included in OpenCV: remember to turn on gPhoto2 option in Cmake.list when building OpenCV.
此时，可以使用我们的数码相机，使用OpenCV中最近包含的libgphoto2库捕获图像：构建OpenCV时，请记住在Cmake.list中打开gPhoto2选项。

@code{.cpp}
  // Open camera number 1, using libgphoto2
  VideoCapture cap1( CAP_GPHOTO2 );
  if( !cap1.isOpened() )
  {
    // check if cam1 opened
    cout << "cam1 not opened!" << endl;
    help();
    return -1;
  }
  // Open camera number 2
  VideoCapture cap2( 1 );
  if( !cap2.isOpened() )
  {
     // check if cam2 opened
     cout << "cam2 not opened!" << endl;
     help();
     return -1;
  }
@endcode

The two cameras must work at the same resolution and must have autofocus option disabled, maintaining the same focus during all acquisition. The projector can be positioned in the middle of the cameras.
两台相机必须以相同的分辨率工作，并且必须禁用自动对焦选项，在所有采集过程中保持相同的对焦。投影机可以放置在摄像机中间。

However, before to proceed with pattern acquisition, the cameras must be calibrated. Once the calibration is performed, there should be no movement of the cameras, otherwise a new calibration will be needed.
但是，在进行图案采集之前，必须先校准摄像机。一旦执行了校准，照相机就不应移动，否则将需要新的校准。

After having connected the cameras and the projector to the computer, cap_pattern demo can be launched giving as parameters the path where to save the images, and the projector's width and height, taking care to use the same focus and cameras settings of calibration.
将照相机和投影仪连接到计算机后，可以启动cap_pattern演示，其中将保存图像的路径以及投影仪的宽度和高度作为参数，并注意使用相同的焦点和照相机校准设置。

At this point, to acquire the images with both cameras, the user can press any key.
此时，要使用两台摄像机获取图像，用户可以按任意键。

@code{.cpp}
  // Turning off autofocus
  cap1.set( CAP_PROP_SETTINGS, 1 );
  cap2.set( CAP_PROP_SETTINGS, 1 );
  int i = 0;
  while( i < (int) pattern.size() )
  {
    cout << "Waiting to save image number " << i + 1 << endl << "Press any key to acquire the photo" << endl;
    imshow( "Pattern Window", pattern[i] );
    Mat frame1;
    Mat frame2;
    cap1 >> frame1;  // get a new frame from camera 1
    cap2 >> frame2;  // get a new frame from camera 2
     ...
  }
@endcode

If the captured images are good (the user must take care that the projected pattern is viewed from the two cameras), the user can save them pressing the enter key, otherwise pressing any other key he can take another shot.
如果捕获的图像质量很好（用户必须注意从两个摄像机观看投影的图案），则用户可以按Enter键保存它们，否则按任何其他键可以再次拍摄。

@code{.cpp}
      // Pressing enter, it saves the output
      if( key == 13 )
      {
        ostringstream name;
        name << i + 1;
        save1 = imwrite( path + "pattern_cam1_im" + name.str() + ".png", frame1 );
        save2 = imwrite( path + "pattern_cam2_im" + name.str() + ".png", frame2 );
        if( ( save1 ) && ( save2 ) )
        {
          cout << "pattern cam1 and cam2 images number " << i + 1 << " saved" << endl << endl;
          i++;
        }
        else
        {
          cout << "pattern cam1 and cam2 images number " << i + 1 << " NOT saved" << endl << endl << "Retry, check the path"<< endl << endl;
        }
      }
@endcode

The acquistion ends when all the pattern images have saved for both cameras. Then the user can reconstruct the 3D model of the captured scene using the *decode* method of *GrayCodePattern* class (see next tutorial).
当所有图案图像都保存到两个摄像机时，采集结束。然后，用户可以使用* GrayCodePattern *类的* decode *方法重建捕获的场景的3D模型（请参阅下一个教程）。
