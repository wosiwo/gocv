# GOCV

本仓库是[官方gocv](https://github.com/hybridgroup/gocv)的fork，原始的README请查看[这里](https://github.com/LKKlein/gocv/blob/lk/README_ORIGIN.md)。  

本fork主要修改了官方gocv接口中，部分与其他语言的结果不一致的API，同时还添加了部分新的API。

## 1. API修改

### 1.1 RotatedRect结构体的修改

- before
    ```C
    // c语言结构体
    typedef struct RotatedRect {
        Contour pts;	
        Rect boundingRect;	
        Point center;	
        Size size;	
        double angle;	
    } RotatedRect;

    // go语言结构体
    type RotatedRect struct {
        Contour      []image.Point
        BoundingRect image.Rectangle
        Center       image.Point
        Width        int
        Height       int
        Angle        float64
    }
    ```

- after
    ```C
    // c语言结构体
    typedef struct RotatedRect {
        Contour pts;
        Rect boundingRect;
        Point2f center;
        Size2f size;
        double angle;
    } RotatedRect;

    // go语言结构体
    type RotatedRect struct {
        Contour      []image.Point
        BoundingRect image.Rectangle
        Center       Point2f
        Width        float32
        Height       float32
        Angle        float64
    }
    ```

这其中主要修改 `center` 和 `size` 两个参数的类型，在C++和Python的OpenCV中，`center` 和 `size` 都是支持`float`的，但是在原版gocv中，直接固定为`int`，这样会导致`RotatedRect`相关的操作会出现偏差。

- 相关API修改

```go
func BoxPoints(rect RotatedRect, pts *Mat)
func MinAreaRect(points []image.Point) RotatedRect
func FitEllipse(points []image.Point) RotatedRect
```

### 1.2 `WarpPerspective`函数参数的修改

C++中`warpPerspective`函数包含7个可选参数，但是在gocv的接口中只给出了4个参数，后3个参数全部固定，因此当后3个参数为其他内容时，会导致计算偏差。  

因此，此处新增一个`WarpPerspectiveWithParams`函数，保留全部7个可调节参数，保证结果的一致性。

- 添加API

```go
func WarpPerspectiveWithParams(src Mat, dst *Mat, m Mat, sz image.Point, flags InterpolationFlags, borderType BorderType, borderValue color.RGBA)
```

## 2. 新增API

除了gocv提供的一些API外，还有一些个人需要的API，这里会逐渐添加，目前添加的API包括

```go
// 矩阵与向量的广播操作，适合图像各通道独立计算
func (m *Mat) AddScalar(value Scalar)
func (m *Mat) SubtractScalar(value Scalar)
func (m *Mat) MultiplyScalar(value Scalar)
func (m *Mat) DivideScalar(value Scalar)
```
