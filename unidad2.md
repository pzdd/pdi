## Processamento Digital da Imagens

### [Início](/index.md) - [Unidade 1](/unidad1.md) - Unidade 2

### Exercício 8.2

O objetivo desta tarefa é implementar um filtro homomórfico para melhorar imagens com iluminação irregular.

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
#include <opencv2/imgproc/imgproc.hpp>

#define RADIUS 20
#define MAX_GAMA_H  100.0
#define MAX_GAMA_L  100.0
#define MAX_D0      100.0
#define MAX_C       100.0

using namespace cv;
using namespace std;

int gama_H, gama_L, d0, c;

int dft_M, dft_N;
Mat filter, tmp;

void on_filter(int, void*);
void deslocaDFT(Mat& image );

int main(int , char**){
  VideoCapture cap;
  Mat imaginaryInput, complexImage, multsp;
  Mat padded, mag;
  Mat image, imagegray;
  Mat_<float> realInput, zeros;
  vector<Mat> planos;

  char key;   // guarda tecla capturada

  cap.open(0);  // abre a câmera default
  if(!cap.isOpened()) return -1;

  // captura uma imagem para recuperar as
  // informacoes de gravação
  cap >> image;

  // identifica os tamanhos otimos para
  // calculo do FFT
  dft_M = getOptimalDFTSize(image.rows);
  dft_N = getOptimalDFTSize(image.cols);

  // realiza o padding da imagem
  copyMakeBorder(image, padded, 0,
                 dft_M - image.rows, 0,
                 dft_N - image.cols,
                 BORDER_CONSTANT, Scalar::all(0));

  // parte imaginaria da matriz complexa (preenchida com zeros)
  zeros = Mat_<float>::zeros(padded.size());

  // prepara a matriz complexa para ser preenchida
  complexImage = Mat(padded.size(), CV_32FC2, Scalar(0));

  // a função de transferência (filtro frequencial) deve ter o
  // mesmo tamanho e tipo da matriz complexa
  filter = complexImage.clone();

  // cria uma matriz temporária para criar as componentes real
  // e imaginaria do filtro ideal
  tmp = Mat(dft_M, dft_N, CV_32F);

  char title[25];
  namedWindow("filtrada", 1);

  sprintf       ( title, "Gama H x %.1lf", MAX_GAMA_H );
  createTrackbar( title, "filtrada",   &gama_H, MAX_GAMA_H, on_filter );

  sprintf       ( title, "Gama L x %.1lf", MAX_GAMA_L );
  createTrackbar( title, "filtrada",   &gama_L, MAX_GAMA_L, on_filter );

  sprintf       ( title, "D0 x %.1lf",  MAX_D0 );
  createTrackbar( title, "filtrada", &d0, MAX_D0, on_filter );

  sprintf       ( title, "c x %.1lf",   MAX_C );
  createTrackbar( title, "filtrada", &c, MAX_C, on_filter );

  on_filter(0, 0);


  for(;;){
    cap >> image;
    cvtColor(image, imagegray, CV_BGR2GRAY);
    imshow("original", imagegray);

    // realiza o padding da imagem
    copyMakeBorder(imagegray, padded, 0,
                   dft_M - image.rows, 0,
                   dft_N - image.cols,
                   BORDER_CONSTANT, Scalar::all(0));

    // limpa o array de matrizes que vao compor a
    // imagem complexa
    planos.clear();
    // cria a compoente real
    realInput = Mat_<float>(padded);
    // insere as duas componentes no array de matrizes
    planos.push_back(realInput);
    planos.push_back(zeros);


    realInput = realInput + Scalar::all(1);
    log(realInput,realInput);

    // combina o array de matrizes em uma unica
    // componente complexa
    merge(planos, complexImage);

    // calcula o dft
    dft(complexImage, complexImage);

    // realiza a troca de quadrantes
    deslocaDFT(complexImage);

    // aplica o filtro frequencial
    mulSpectrums(complexImage,filter,complexImage,0);

    // limpa o array de planos
    planos.clear();
    // separa as partes real e imaginaria para modifica-las
    split(complexImage, planos);

    // usa o valor medio do espectro para dosar o ruido

    // recompoe os planos em uma unica matriz complexa
    merge(planos, complexImage);

    // troca novamente os quadrantes
    deslocaDFT(complexImage);

    // calcula a DFT inversa
    idft(complexImage, complexImage);

    // limpa o array de planos
    planos.clear();

    // separa as partes real e imaginaria da
    // imagem filtrada
    split(complexImage, planos);

    // normaliza a parte real para exibicao
    normalize(planos[0], planos[0], 0, 1, CV_MINMAX);
    exp(planos[0],planos[0]);
    normalize(planos[0], planos[0], 0, 1, CV_MINMAX);
    imshow("filtrada", planos[0]);

    key = (char) waitKey(10);
    if(key == 27) break;

  }
  return 0;
}

void on_filter(int, void*){

  float _d0 =    ((float) 2*d0/ (float) MAX_D0) * sqrt( dft_M*dft_M + dft_N*dft_N )/2;
  float _gamaL = ((float) 10*gama_L)/ (float) MAX_GAMA_L;
  float _gamaH = ((float) 10*gama_H)/ (float) MAX_GAMA_H;
  float _c =     ((float) 10*c)/ (float) MAX_C;

  cout << "d0: " << _d0 << "  L: " << _gamaL << "  H: "  << _gamaH << "  c: " << _c << endl;

  double n1, n2;
  double n = 0.0;
  double dGamas = _gamaH - _gamaL;
  double d02 = _d0*_d0;

  for(int i=0; i<dft_M; i++)
  {
    for(int j=0; j<dft_N; j++)
    {
        n1 = (i - dft_M/2)*(i - dft_M/2);
        n2 = (j - dft_N/2)*(j - dft_N/2);
        n = _c * (n1 + n2);

        tmp.at<float>(i,j) = dGamas * (1 - exp(-n/d02)) + _gamaL;
    }
  }

  Mat comps[]= {tmp, tmp};
  merge(comps, 2, filter);
}

// troca os quadrantes da imagem da DFT
void deslocaDFT(Mat& image ){
  Mat aux, A, B, C, D;

  // se a imagem tiver tamanho impar, recorta a regiao para
  // evitar cópias de tamanho desigual
  image = image(Rect(0, 0, image.cols & -2, image.rows & -2));
  int cx = image.cols/2;
  int cy = image.rows/2;

  // reorganiza os quadrantes da transformada
  // A B   ->  D C
  // C D       B A
  A = image(Rect(0, 0, cx, cy));
  B = image(Rect(cx, 0, cx, cy));
  C = image(Rect(0, cy, cx, cy));
  D = image(Rect(cx, cy, cx, cy));

  A.copyTo(aux);  D.copyTo(A);  aux.copyTo(D); // A <-> D
  C.copyTo(aux);  B.copyTo(C);  aux.copyTo(B); // C <-> B
}

```

![Figura 4: Resultado](/images/tela4.png)

### Exercício 11.1

Utilizando os programas exemplos/canny.cpp e exemplos/pontilhismo.cpp como referência, implemente um programa cannypoints.cpp. A idéia é usar as bordas produzidas pelo algoritmo de Canny para melhorar a qualidade da imagem pontilhista gerada. A forma como a informação de borda será usada é livre. Entretanto, são apresentadas algumas sugestões de técnicas que poderiam ser utilizadas:

Imagem original utilizada:

![Figura 5: Imagem Original](/images/tela5.jpg)

Imagem resultante:

![Figura 6: Resultado](/images/tela6.jpg)

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <fstream>
#include <iomanip>
#include <vector>
#include <algorithm>
#include <numeric>
#include <ctime>
#include <cstdlib>

using namespace std;
using namespace cv;

#define STEP 5
#define JITTER 3
#define DIR_IMAGEM "big_ben.jpg"

int top_slider = 10;
int top_slider_max = 200;
int rad_slider = 10;
int rad_slider_max = 200;
int state;
char TrackbarName[50];

Mat image, border,points;
int width, height;

void on_trackbar(int, void*){
  vector<int> yrange;
  vector<int> xrange;
  Mat frame;
  int x, y,gray;

  srand(time(0));

  xrange.resize(height/STEP);
  yrange.resize(width/STEP);

  iota(xrange.begin(), xrange.end(), 0);
  iota(yrange.begin(), yrange.end(), 0);

  for(uint i=0; i<xrange.size(); i++){
    xrange[i]= xrange[i]*STEP+STEP/2;
  }
  for(uint i=0; i<yrange.size(); i++){
    yrange[i]= yrange[i]*STEP+STEP/2;
  }

  points = Mat(height, width, CV_8U, Scalar(255));

  random_shuffle(xrange.begin(), xrange.end());
  for(auto i : xrange){
    random_shuffle(yrange.begin(), yrange.end());
    for(auto j : yrange){
      x = i+rand()%(2*JITTER)-JITTER+1;
      y = j+rand()%(2*JITTER)-JITTER+1;
      gray = image.at<uchar>(x,y);
      circle(points, Point(y,x), (rad_slider/50)+1, CV_RGB(gray,gray,gray), -1, CV_AA);
    }
  }
  Canny(image, border, top_slider, 3*top_slider);
  for(int i=0;i<height;i++){
    for(int j=0;j<width;j++){
      if(border.at<uchar>(i,j)==255){
        gray = image.at<uchar>(i,j);
      circle(points, Point(j,i), (rad_slider/50)+1, CV_RGB(gray,gray,gray), -1, CV_AA);
    }
  }
}
  imwrite(DIR_IMAGEM, points);
  imshow("CannyPointilhismo", points);
}

int main(int argc, char**argv){

  image= imread(DIR_IMAGEM,CV_LOAD_IMAGE_GRAYSCALE);
  width=image.size().width;
  height=image.size().height;
  sprintf( TrackbarName, "Bordas");
  namedWindow("CannyPointilhismo",WINDOW_AUTOSIZE);
  createTrackbar( TrackbarName, "CannyPointilhismo", &top_slider, top_slider_max, on_trackbar);
  on_trackbar(top_slider, 0 );
  sprintf( TrackbarName, "Raio");
  createTrackbar( TrackbarName, "CannyPointilhismo", &rad_slider, rad_slider_max, on_trackbar);
  on_trackbar(rad_slider, 0 );

  waitKey();
  return 0;
}

```
