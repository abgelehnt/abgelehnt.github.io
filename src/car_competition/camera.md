# 信标篇

备赛前期，我们整了各种高级算法，比如：自适应卷积，BPNN，大津法等，但是过滤阳光的效果一般。后来我们决定相信南信大的场地，索性不做阳光算法。

提取信标的流程为：得到一幅图像->进行二值化->划分天地分界线->在地面区域标记连通域->提取最大的连通域为信标并获取信标的各种信息。

## 固定阈值二值化

``` c
void threshold(uint8_t *binary_image_data, uint8_t *output_data, int width, int height, int thres) {
	for (int y = 0; y < height; y++) {
		for (int x = 0; x < width; x++) {
			output_data[x + y * width] = binary_image_data[x + y * width] > thres ? 255 : 0;
		}
	}
}
```

## 种子填充法标记连通域

``` c
int QueueArray[300];
int QueueFirst;
int QueueLast;

void PushQueue(int data) {
	QueueArray[(QueueLast++) % 300] = data;
}

int PopQueue() {
	if (QueueLast == QueueFirst)
		return -1;
	else
		return QueueArray[(QueueFirst++) % 300];
}

const int NeighborDirection[8][2] = {{0,  1},
									 {1,  1},
									 {1,  0},
									 {1,  -1},
									 {0,  -1},
									 {-1, -1},
									 {-1, 0},
									 {-1, 1}};

void SearchNeighbor(unsigned char *bitmap, int width, int height, unsigned char *labelmap,
					int labelIndex, int pixelIndex) {
	int searchIndex, i, length;
	labelmap[pixelIndex] = labelIndex;
	length = width * height;
	for (i = 0; i < 8; i++) {
		searchIndex = pixelIndex + NeighborDirection[i][0] * width + NeighborDirection[i][1];
		if (searchIndex > 0 && searchIndex < length &&
			bitmap[searchIndex] == 255 && labelmap[searchIndex] == 0) {
			labelmap[searchIndex] = labelIndex;
			PushQueue(searchIndex);
		}
	}
}

int ConnectedComponentLabeling(unsigned char *bitmap, int width, int height, unsigned char *labelmap) {
	QueueFirst = 0;
	QueueLast = 0;
	int cx, cy, index, popIndex, labelIndex = 0;
	memset(labelmap, 0, width * height);
	for (cy = 1; cy < height - 1; cy++) {
		for (cx = 1; cx < width - 1; cx++) {
			index = cy * width + cx;
			if (bitmap[index] == 255 && labelmap[index] == 0) {
				labelIndex++;
				SearchNeighbor(bitmap, width, height, labelmap, labelIndex, index);

				popIndex = PopQueue();
				while (popIndex > -1) {
					SearchNeighbor(bitmap, width, height, labelmap, labelIndex, popIndex);
					popIndex = PopQueue();
				}
			}
		}
	}
	return labelIndex;
}
```

## 划分天地分界线

虽然我们不对阳光做处理，但是日光灯对识别的影响是必定存在的。

具体思路为：

1. 在标定分界线之前，我们需要已经写好连通域算法并对全图进行连通域标记。
2. 在一个距离灯尽可能远的地方打印出当前灯在摄像头的y坐标和当前陀螺仪的俯仰角数据，并使用串口发送到电脑上。
3. 使用MATLAB拟合出灯的y坐标和陀螺仪俯仰角的函数关系。
4. 根据实时陀螺仪俯仰角计算出y坐标值，小于此值为天，大于此值的为地。
5. 忽略天部分的图像，只对地部分标记连通域以减少计算量。

## 找出正确的连通域

得到地部分的连通域后，我们可以简单的取最大的连通域作为信标，我们可以得到信标的大小、位置等信息。

``` c
left = MT9V03X_W, right = 0, up = MT9V03X_H, down = 0;
int max_light = 0, average_light = 0, gravity_x = 0, gravity_y = 0, light_x = 0, light_y = 0, side_x = 0, side_y = 0, counter = 0;
for (int j = 0; j < MT9V03X_H; j++) {
	for (int k = 0; k < MT9V03X_W; k++) {
		if (firstPass[j][k] == i) {
			counter++;
			if (k < left) {
				left = k;
			}
			if (k > right) {
				right = k;
			}
			if (j < up) {
				up = j;
			}
			if (j > down) {
				down = j;
			}
			if (mt9v03x_image[j][k] > max_light) {
				max_light = mt9v03x_image[j][k];
			}
			average_light += mt9v03x_image[j][k];
			gravity_x += mt9v03x_image[j][k] * j;
			gravity_y += mt9v03x_image[j][k] * k;
			light_x += mt9v03x_image[j][k];
			light_y += mt9v03x_image[j][k];
			if (k != 0 && firstPass[j][k - 1] != i) {
				side_x++;
			}
			if (j != 0 && firstPass[j - 1][k] != i) {
				side_y++;
			}
		}
	}
}
```

## 调试镜头

- 如果感觉得到的图像明显偏移，可以看看镜座是否在镜头的正中。
- 如果图像明显模糊，可以旋转镜头调焦距。
