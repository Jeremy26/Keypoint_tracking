# Writeup
The purpose of this writeup is to explain how I addressed each point.

### Data Buffer
The data buffer is a vector that contains 10 images. Not more.
When the buffer size exceeds this number (here 10), we remove the most ancient element and push the new frame on top of the list.
```C++
        if (dataBuffer.size() == dataBufferSize){
            dataBuffer.erase(dataBuffer.begin()); 
        }
        dataBuffer.push_back(frame);
```

### Keypoints
1. *Detection*
To find keypoints, we use detectors such as HARRIS, FAST, BRISK, ORB, AKAZE and SIFT. We can select any detector we like by replacing the string variable. Then, we use OpenCV to implement each detector.
Here is an example.
ShiTomasi and Harris detection were built through the course. Others use default functions.
SIFT detector requires a different library.
```C++
  if (detectorType.compare("FAST") == 0)
    {
        detector = cv::FastFeatureDetector::create();
    }
```

2. *ROI*
A Region of Interest is performed to remove some unnecessary points and focus on the preceding  vehicle.
 We use the openCV::Rect function. If a keypoint is within this rectangle, we erase it. 
```C++
        cv::Rect vehicleRect(535, 180, 180, 150);
        if (bFocusOnVehicle)
        {
            auto it = keypoints.begin();
            while(it!= keypoints.end()){
                int keyptx = static_cast<int>((*it).pt.x);
                int keypty = static_cast<int>((*it).pt.y);
                if (keyptx<vehicleRect.x || keyptx>vehicleRect.x+vehicleRect.width || keypty<vehicleRect.y || keypty>vehicleRect.y+vehicleRect.height)

                {
                    keypoints.erase(it);
                }
                else{
                    ++it;
                }
            }
```

### Descriptors
1. *Selection*
Once the detection method is selected, we need to use a description method.
Descriptors can be BRIEF, ORB, AKAZE, or SIFT.
They are implemented similarly to detectors, using an if/then statement.
They have some particularities: 
```C++
else if (descriptorType.compare("SIFT") == 0)
    {
        extractor = cv::xfeatures2d::SIFT::create();
    }
```
2. *Matching*
FLANN matching has been implemented using OpenCV as well as K-Nearest-Neighbor.
However, we can choose between Brute Force and Flann.
Detectors/Descriptors will not work similarly.
For example, AKAZE descriptors only work with AKAZE detectors.
If we are using SIFT as a descriptor, we cannot use the Hamming distance as a comparison. Since the descriptor SIFT is a continuous vector, we cannot use the HAMMING distance and need to specify L2_NORM; when using Brute Force matching.
As OpenCV bug was also found and required to modify  the type was 32_F for descriptors whose type were different.
Here is the function for better understanding.
```C++
    if (matcherType.compare("MAT_BF") == 0)
    {
        int normType = descriptorType.compare("DES_BINARY") == 0 ? cv::NORM_HAMMING : cv::NORM_L2;
        matcher = cv::BFMatcher::create(normType, crossCheck);
    }
    else if (matcherType.compare("MAT_FLANN") == 0)
    {
        // OPENCV 'BUG'
       if (descSource.type()!=CV_32F || descRef.type()!=CV_32F)
       {
           descSource.convertTo(descSource, CV_32F);
           descRef.convertTo(descRef, CV_32F);
       }
        matcher = cv::DescriptorMatcher::create(cv::DescriptorMatcher::FLANNBASED);
    }
```

### Performance
All performance metrics and explanations can be found on the README page.
