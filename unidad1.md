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

Esta tarefa trata-se de trocar as regiões de uma imagem.

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

int main(int argc, char** argv) {
	Mat image, newImage;
	Size size;
	int newHeight, newWidth;

	image = imread("biel.png", CV_LOAD_IMAGE_GRAYSCALE);
	image.copyTo(newImage);
	size = image.size();

	newHeight = size.height/2;
	newWidth = size.width/2;
	
	//trocando os quadrantes em diagonal na imagem
	Mat b1 = image( Rect(0, 0, newHeight, newWidth) );
	Mat b2 = image( Rect(newHeight, 0, newHeight, newWidth) );
	Mat b3 = image( Rect(0, newWidth, newHeight, newWidth) );
	Mat b4 = image( Rect(newHeight, newWidth, newHeight, newWidth) );

	b4.copyTo( newImage( Rect(0, 0, newHeight, newWidth) ) ); // 1
	b3.copyTo( newImage( Rect(newHeight, 0, newHeight, newWidth) ) ); // 2
	b2.copyTo( newImage( Rect(0, newWidth, newHeight, newWidth) ) ); // 3
	b1.copyTo( newImage( Rect(newHeight, newWidth, newHeight, newWidth) ) ); // 4

	imshow("image", newImage);
	waitKey();

	return 0;
}
```
![Figura 2: Resultado](/images/tela2.png)

### Exercício 4.2

O objetivo desse exercicio é identicar em uma imagem regiões com ou sem buracos internos presentes na cena.
Os objetos que tocam as bordas foram desconsiderados, pois não dão informaçes suficientes para fazer afirmaçes sobre o mesmo.

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace cv;
using namespace std;

int main(int argc, char** argv){
	Mat image, mask;
	int width, height;
	int objetos = 0;
	int buracos = 0;

	CvPoint p;
	image = imread("bolhas.png",CV_LOAD_IMAGE_GRAYSCALE);

	if(!image.data)
	{
		cout << "[erro]\n";
		return(-1);
	}

	width  = image.size().width;
	height = image.size().height;

	p.x = 0;
	p.y = 0;

	//Primeiro passo
  //Elimando objetos das bordas
	for (int i = 0; i < width; i++){
		image.at<uchar>(i, 0) = 255;
		image.at<uchar>(i, height-1)= 255;
	}
	for (int i = 0; i < height; i++){
		image.at<uchar>(0, i) = 255;
		image.at<uchar>(width-1, i) = 255;
	}
	floodFill(image, p, 0);

  //alterando o background
	floodFill(image, p, 244);

	//Segundo passo
  //Contando objetos
	for(int i = 0; i < height; i++){
		for(int j = 0; j < width; j++){
			p.x = j;
			p.y = i;
	  	if(image.at<uchar>(i,j) == 255) floodFill(image, p, ++objetos);
	  	if(image.at<uchar>(i,j) == 0) floodFill(image, p, 244 - (++buracos));
		}
	}

	cout << "Objetos: "<< objetos << "\nBuracos: " << buracos << endl;

	imshow("image", image);
	imwrite("bolhas_saida.png", image);
	waitKey();
	return 0;
}

```

![Figura 3: Resultado](/images/tela3.png)
