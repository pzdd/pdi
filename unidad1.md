## Processamento Digital da Imagens

### [Início](/index.md) - Unidade 1

### Exercício 3.2 A

O objetivo desse programa é selecinar uma região da imagem e substituir essa região pelo seu negativo. O negativo de uma imagem é obtido quando se subtitui o valor de cada pixel pelo seu complemento.

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

struct P {
	int x;
	int y;
};

int main(int argc, char** argv) {

	if(argc != 2) {
		cout << "[exe] ./your_program_name image.png\n";
		return -1;
	}

	P p1, p2;
	Mat image;
	Size size;

	image = imread(argv[1], CV_LOAD_IMAGE_GRAYSCALE);

	size = image.size();

	{
		cout << "Negativar uma região\n";
		cout << "Informe dois pontos da região que será negativada...\n";
		cout << "[INFO]:\n";
		cout << "height = " << size.height << " width = " << size.width << endl;

		cout << "Informe o valor de x do Ponto 1: ";
		cin >> p1.x;
		cout << "Informe o valor de y do Ponto 1: ";
		cin >> p1.y;

		cout << "Informe o valor de x do Ponto 2: ";
		cin >> p2.x;
		cout << "Informe o valor de y do Ponto 2: ";
		cin >> p2.y;
	}

	//substituindo o valor dos pixels pelo seu complemento
	for(int x=p1.x; x<=p2.x; x++) {
		for(int y=p1.y; y<=p2.y; y++) {
			image.at<uchar>(x,y)= 255 - image.at<uchar>(x,y);
		}
	}

	cv::imshow("image", image);
	cv::waitKey();

  return 0;
}

```

![Figura 1: Resultado](/images/tela1.png)



### Exercício 3.2 B

### Exercício 4.2

