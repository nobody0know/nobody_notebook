# 点云库PCL学习

## 点云输入输出

### 载入pcd文件

```cpp
#include<iostream>
#include<pcl/io/pcd_io.h>
#include<pcl/point_types.h>
using namespace std;
using namespace pcl;
int main(int argc,char **argv)
{
    PointCloud<PointXYZ>::Ptr cloud(new PointCloud<PointXYZ>);
    if(pcl::io::loadPCDFile<pcl::PointXYZ>("./test_pcd.pcd",*cloud)==-1)
    {
        PCL_ERROR("not read pcd\n");
        return -1;
    }
    std::cout<<"load "
             << cloud->width * cloud->height
             <<"data points from test_pcd.pcd with the following fields:"
             <<std::endl;
    for (size_t i = 0; i < cloud->points.size(); i++)
    {
        std::cout << "    " << cloud->points[i].x
                  << " " << cloud->points[i].y
                  << " " << cloud->points[i].z << std::endl;       
    }
    return 0;
}
```

## 输出pcd文件

```cpp
#include <iostream>
#include <pcl/io/pcd_io.h>
#include <pcl/point_types.h>

int main(int argc, char **argv)
{
    pcl::PointCloud<pcl::PointXYZ> cloud;

    // Fill in the cloud data
    cloud.width = 5;
    cloud.height = 1;
    cloud.is_dense = false;
    cloud.points.resize(cloud.width * cloud.height);

    for (auto &point : cloud)
    {
        point.x = 1024 * rand() / (RAND_MAX + 1.0f);
        point.y = 1024 * rand() / (RAND_MAX + 1.0f);
        point.z = 1024 * rand() / (RAND_MAX + 1.0f);
    }

    pcl::io::savePCDFileASCII("test_pcd.pcd", cloud);
    std::cerr << "Saved " << cloud.size() << " data points to test_pcd.pcd." << std::endl;

    for (const auto &point : cloud)
        std::cerr << "    " << point.x << " " << point.y << " " << point.z << std::endl;

    return (0);
}
```
