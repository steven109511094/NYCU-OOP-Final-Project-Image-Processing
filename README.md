# NYCU-OOP-Final-Project-Image-Processing
Last Updated: 2025/05/07

:::danger
請妥善使用E3平台上的討論區發問!
建議可以開啟訂閱!(一旦有人發問或回答，系統會自動寄信到你的信箱通知)  
非不得已請不要寄信給助教!(如:在討論區發問過一段時間了，但沒有助教理你QAQ)
要寄信的話，麻煩同時寄給四位助教，我們會盡快回覆!
寄信給助教詢問前，請先看一下問題有沒有被問過!  
:::
## 簡介
假想今天我們想要將一串文字隱藏起來，除了把它背起來銷毀，和把它埋到某個不知名的地方外，還有沒有什麼辦法呢?
沒錯!就是加密，但為了讓題目更有趣點，我們融入了影像處理，也就是把文字加密進圖像當中!
![image](https://hackmd.io/_uploads/SyvlHiPgxe.png)

如圖所示，我們在原來的照片中隱藏了"I love OOP"這串文字，但卻沒有讓圖片有太大的變動。(畢竟如果圖片變得太奇怪，可能就會被人看出端倪?)
當然了，我們也要能夠從加密完的圖片中，解碼出原來的字串。
若是只有這樣，題目難免有些單調，因此我們也鼓勵同學們上網搜尋與實作各種有關影像處理、文字加密的算法。只要demo時展示功能並附註於結報中，助教們會依照內容豐富度與難度進行額外加分!

## Preface: What is Digital Image?
數位影像是電腦處理的視覺資訊表現形式，與傳統模擬影像不同，它由離散的像素構成，每個像素包含特定的顏色和亮度資訊，而這些像素的排列形成了影像的整體視覺內容。

ex：8 位元灰階影像
*    位元深度：
        "8 位元"指的是每個像素的位深度。在 8 位元的灰階影像中，每個像素有 2^8（256）種可能的值，範圍從 0 到 255。
        
*    灰階(有多黑/有多白)：
不同於彩色影像，灰階影像包含各種灰色調。在 8 位元灰階影像中，每個像素代表不同的強度水平，其中 0 為黑色，255 為白色，中間有各種灰色調。
![image](https://hackmd.io/_uploads/ry0phvoW0.png)
Ref: [https://processing.org/tutorials/color]

![image](https://hackmd.io/_uploads/SkegTwj-0.png)

:::success    
因此，在電腦圖形中表示灰階影像是非常簡單的:使用一個二維陣列，裡面每個數值皆落在 0 到 255 的範圍。若是彩色圖片，則將其二維陣列擴展為有RGB三個channel的三維陣列即可。
:::
![image](https://hackmd.io/_uploads/B1IwawiW0.png)

## Step 1: Play around data loader class
圖片在存成jpg/png是有經過壓縮加密過的，若是學員要直接讀取圖片檔，會有不少問題。因此提供data_loader使學員可以直接得到圖片的像素矩陣及長寬，來進行後續的運算及操作。

以下是data_loader的interface:
```c=
class Data_Loader{

public:
    Data_Loader();
    ~Data_Loader();
    int **Load_Gray(string filename, int *w, int *h);
    int ***Load_RGB(string filename, int *w, int *h);
    void Dump_Gray(int w, int h, int **pixels, string filename);
    void Dump_RGB(int w, int h, int ***pixels, string filename);
    void Display_Gray_X_Server(int w, int h, int **pixels);
    void Display_RGB_X_Server(int w, int h, int ***pixels);
    void Display_Gray_ASCII(int w, int h, int **pixels);
    void Display_RGB_ASCII(int w, int h, int ***pixels);
    void Display_Gray_CMD(string filename);
    void Display_RGB_CMD(string filename);
    bool List_Directory(string directoryPath, vector<string> &filenames);

private:
    bool File_Exists(const string &filename);
};
```

這些interface的usage:
*    Load:
        ```c=
        int **Load_Gray(string filename, int *w, int *h);
        int ***Load_RGB(string filename, int *w, int *h);
        ```
        給定圖片的路徑，並且設定w與h，最後回傳一個二維(三維)陣列

        :::danger
        Load回傳的dynamic allocate memory需要由學員自己負責刪除，我們會使用valgrind來檢查學員的程式，確保學員對每一個他們new出來的memory負責。
        :::

*    Dump:
        ```c=
        void Dump_Gray(int w, int h, int **pixels, string filename);
        void Dump_RGB(int w, int h, int ***pixels, string filename);
        ```
        給定輸出的w,h,要輸出的二維(三維)陣列與要輸出的圖片檔名，會將圖片輸出成*jpg/*png。

data_loader提供了三種介面來展示圖片: 
*    1. X_Server
        使用moba_xterm中的Xserver來跳出視窗來顯示圖片。
        ```c=
        void Display_Gray_X_Server(int w, int h, int **pixels);
        void Display_RGB_X_Server(int w, int h, int ***pixels);
        ```
        
        ![image](https://hackmd.io/_uploads/BJZsWdjZC.png)
 
        :::danger
        因為valgrind在mem check時，無法辨認此function的部分macro expansion，因此在做memory leak check時，請不要使用此function。
        :::
 
*    2. ASCII ART
        使用" .-+#@"來表示圖片中的明暗程度，將圖片以符號的形式印在terminal。
        
        ```c=
        void Display_Gray_ASCII(int w, int h, int **pixels);
        void Display_RGB_ASCII(int w, int h, int ***pixels);
        ```
        ![image](https://hackmd.io/_uploads/HJvPGdiWC.png)

*    3. catimg 
        將圖片本身直接印在terminal，支援灰階及彩色圖片。

        ```c=
        void Display_Gray_CMD(string filename);
        void Display_RGB_CMD(string filename);
        ```

        ![image](https://hackmd.io/_uploads/r1FFM_s-R.png)

        :::info
        該function僅吃已存在的圖片檔的檔名，並把圖片印在terminal，請搭配前面所提到的dump使用。
        :::

*    filename iterator

        因為後面會需要學員處理多張照片，這邊提供一個method，來將某資料夾下的所有檔案名稱存進filenames的vector中。

        ```c=
        bool List_Directory(string directoryPath, vector<string> &filenames);
        ```

## Step 2: Construct image inheritance and polymorphism

透過base class Image，來讓gray_image及rgb_image繼承，來讓同學練習繼承多型及virtual function。需要將第一步所提及的data_loader與這些class整合。來實現load image/dump image/display image，等基礎功能。

繼承關係如下圖:
![image](https://hackmd.io/_uploads/HJGOMFs-R.png)


:::spoiler 點選展開繼承詳細指示
*    Base class: Image

        Data member:  
        *    int w(protected)
        *    int h(protected)
        *    Data_Loader data_loader(protected)
                :::info
                data_loader要讓所有image class來共用同一個data_loader，請查詢c++ keyword: static variable
                :::
        
        Member function(all public):
        *    Constructor/Destructor
        *    int get_w()
        *    int get_h()
        
        Pure virtual function(let derived class override, all public):
        *    bool LoadImage(string filename)
        *    void DumpImage(string filename)
        *    void Display_X_Server()
        *    void Display_ASCII()
        *    void Display_CMD()

*    Derived class: GrayImage (public inheritance Image)

        Data member:  
        *    int **pixels(private)

        Member function(all public):
        *    Constructor/Destructor

        Override Base class vitual function(all public):
        *    bool LoadImage(string filename)
        *    void DumpImage(string filename)
        *    void Display_X_Server()
        *    void Display_ASCII()
        *    void Display_CMD()
        
*    Derived class: RGBImage (public inheritance Image)

        Data member:  
        *    int ***pixels(private)

        Member function(all public):
        *    Constructor/Destructor

        Override Base class vitual function(all public):
        *    bool LoadImage(string filename)
        *    void DumpImage(string filename)
        *    void Display_X_Server()
        *    void Display_ASCII()
        *    void Display_CMD()
:::



:::info
學員需要將Step1所提到的data_loader所提供的method整合進這些class中，並且支援polymorphism中的dynamic binding(late binding/run time polymorphism)。
:::

ex:
```c=
Image *img1 = new GrayImage();
img1->LoadImage("Image-Folder/lena.jpg");
img1->DumpImage("img1.jpg");
img1->Display_X_Server();
img1->Display_CMD();


Image *img2 = new RGBImage();
img2->LoadImage("Image-Folder/lena.jpg");
img2->DumpImage("img2.jpg");
img2->Display_X_Server();
img2->Display_CMD();
```
:::info
1. img1 and img2 are pointers to base class Image.  
2. GrayImage and RGBImage are derived classes of Image.  
3. The functions LoadImage, DumpImage, Display_X_Server, and Display_CMD are all virtual functions declared in the Image base class and overridden in the derived classes.
:::


## Step 3: Bit-field with image filter design
一般圖片在做影像處理演算法時，大多會通過許多次的影像增強演算法或是降躁銳化。因此在這個部分，我們要求學生實作出4種指定的簡單影像處理演算法，並透過使用bit_field的方式調用。

*    Bit field介紹:
```c=
//using bitfield to not to force user to passing all of the arguments
//using bitwise or to passing the options
//using bitwise and to get the info of the bitfield

#include <stdio.h>
#include <stdint.h>

#define CASE_ONE    0b00000001
#define CASE_TWO    0b00000010
#define CASE_THREE  0b00000100
#define CASE_FOUR   0b00001000


//using bitwise and to track whtat is the user's option
void loadCase(int8_t option){
    if(option & CASE_ONE)
        printf("Case 1 detected\n");
    if(option & CASE_TWO)
        printf("Case 2 detected\n");
    if(option & CASE_THREE)
        printf("Case 3 detected\n");
    if(option & CASE_FOUR)
        printf("Case 4 detected\n");
    printf("\n");
    printAndResult(option);
}

int main(){
    //test1:
    uint8_t option = 0b00001001;
    printf("test1:\n");
    loadCase(option);

    //test2:
    printf("test2:\n");
    loadCase(CASE_ONE | CASE_TWO);

    //test3:
    printf("test3:\n");
    loadCase(CASE_ONE | CASE_TWO | CASE_THREE | CASE_FOUR);
    return 0;
}
```
:::info
1. 透過`bitwise or` 來load不同的option
2. 透過`bitwise and` 來確認某個option是否有被enable
:::


以下列出本題指定的4個常見影像處理演算法:

*    Horizontal Flip(水平翻轉)
![image](https://hackmd.io/_uploads/Byb1hCDgxg.png)

*    Mosaic filter(對照片打碼)

![螢幕擷取畫面 2025-05-07 030210](https://hackmd.io/_uploads/BkLM6Cwgle.jpg)

*    Gaussian filter(降噪並模糊化)
![image](https://hackmd.io/_uploads/BJVEpRDgge.png)

*    Laplacian filter(影像銳化)
![螢幕擷取畫面 2025-05-07 030433](https://hackmd.io/_uploads/BkSIpRvlex.jpg)

:::info
預期同學實作任意四種以上的圖片影像處理的演算法，並透過bit field的概念來決定要enable哪幾個演算法。
:::

## Step 4: Image encryption

給定一字串，設法在不大幅變動該圖片的前提下，將該字串的資訊存入圖片(可自行準備)，並能夠解碼出來。

學生需要撰寫一個class ImageEncryption，並且與前面所建立好關係的Image繼承鍊及data loader整合，撰寫image encryption的演算法，並提供函式介面給使用者。
*    encode: 傳入欲加密的字串與儲存的圖像路徑，而後回傳一個指標指向加密後的彩圖物件。
*    decode: 傳入一個指標指向加密後的彩圖物件，而後回傳解碼出來的字串。

參考方法: LSB strategy
由於微小R/G/B值的變動並不容易被人類的肉眼察覺，因此我們可以將資訊存在不同pixel channel的LSB(Least Significant Bit)，儲存順序可以依照raster-scan order或是其他遍歷法都可以。而還原方式也非常簡單，只要依照儲存順序將位元重新組合回來即可。
![image](https://hackmd.io/_uploads/SknJIJ_xle.png)

:::info
**LSB strategy**
1. 將字串依ASCII轉換成二進位序列。
2. 依raster-scan order或其他順序，將獲得的二進位序列存入pixel中R/G/B等channel的LSB。
3. 將上述步驟倒過來即為解碼。
:::
![image](https://hackmd.io/_uploads/HkoZFkdlle.png)

:::warning
學員可以在必要的情況下，針對class image gray_image rgb_image增加operator overloading/resize/crop等method，來協助實作image encryption的部分。
:::

## Driven Code for Step1~4
希望學員能夠設計一個清晰且優雅的使用者流程讓上面Step1~4與您所設計的加分項目都可以被demo到。若是單一一個main.cpp不夠您demo所有的功能，您可以自行設計其他driven code的檔案，並且更改`Makefile`中相關編譯的dependency與方法。

    
## Project Setup
```bash=
# login 140.113.201.197 only
$ git clone https://github.com/steven109511094/NYCU-OOP-Final-Project-Image-Processing.git
$ cd NYCU-OOP-Final-Project-Image-Processing/

# install the third party package
$ make install

# start programing your final project
# finish *.h in inc/ & *.cpp in src & main.cpp

# compile
$ make               # default
$ make VERBOSE=1     # check out what make actually do
$ make -j            # compile in parallel (save time, suggest)

# run your program
$ ./Image_Processing

# Dynamic memory check (Need to disable the Display_X_Server...)
$ make check
```
## Project Structure

```bash=
# show出project structure
$ tree -L 2
```
                    ├── Data-Loader(處理image I/O)
                    │   ├── data_loader.cpp
                    │   └── data_loader.h
                    ├── data_loader_demo.cpp(示範如何使用data_loader Step1)
                    ├── Image-Folder(放圖片的地方)
                    │   ├── 1-1.jpg
                    │   ├── 1-2.jpg
                    │   ├── 2-1.jpg
                    │   ├── 2-2.jpg
                    │   ├── 3-1.jpg
                    │   ├── 3-2.jpg
                    │   ├── 4-1.jpg
                    │   ├── 4-2.jpg
                    │   ├── lena.jpg
                    │   └── truck.png
                    ├── inc (put your header here)
                    │   ├── bit_field_filter.h
                    │   ├── gray_image.h
                    │   ├── image_encryption.h
                    │   ├── image.h
                    │   └── rgb_image.h
                    ├── LICENSE
                    ├── main.cpp(Driven code)
                    ├── Makefile
                    ├── README.md
                    ├── scripts
                    │   └── clone_env.sh
                    ├── src (put your implementation here)
                    │   ├── bit_field_filter.cpp
                    │   ├── gray_image.cpp
                    │   ├── image.cpp
                    │   ├── image_encryption.cpp
                    │   └── rgb_image.cpp
                    └── third-party(第三方開源圖片套件)
                        ├── catimg
                        ├── CImg
                        └── libjpeg

將class header interface放在inc folder內部，並且將source code的實作放在src folder內部，makefile會自動去識別dependency，並且在您對某些檔案進行修改後，僅編譯需要重新編譯之檔案，不會整份project重新編譯一次，如此一來再搭配上parallel compile，讓您再開發上能夠節省不少時間。

## Bonus
*    Upload project to github。(需附上project repo) [ref link](https://github.com/twtrubiks/Git-Tutorials)  
        
比較完整的git/github教學:
<iframe width="560" height="315" src="https://www.youtube.com/embed/FKXRiAiQFiY?si=tNPLQ83SkLfbQT6x" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


*    More image filters
        *    fisheye filter
        *    cold/warm adjustment
        *    luminance enhancement

*    Other encryption/decryption methods
        *    XOR encryption
        *    Caesar Cipher
        *    Substitution Cipher

*    Photo Mosaic(小圖組大圖)
        *    [2024-NYCU-OOP-Final-Project](https://github.com/coherent17/NYCU-OOP-Final-Project)
        *    Photo Mosaic with only 1 picture [IEEE paper](https://ieeexplore.ieee.org/document/7965140)
        ![image](https://hackmd.io/_uploads/r171Wpq-C.png)
        *    Parallel Algorithm Implementation(MPI/pthread/cuda): NTHU PP周志遠教授

        
:::info
以上為助教們推薦的幾個加分題的方向，學生不需要侷限在這些題目中，可以自行發想有趣的題目。
:::

## Submission
*    時間: 未定
*    繳交檔案: `Student_ID.tar` & `Student_ID.pdf`
        ```bash=
        # 產生壓縮檔
        $ ls # 確認已跳到NYCU-OOP-Final-Project-Image-Processing外面
        $ tar cvf Student_ID.tar NYCU-OOP-Final-Project-Image-Processing/
        $ ls # 產生Student_ID.tar -> submit to e3

        # 解壓縮
        $ tar xvf Student_ID.tar
        ```
:::info
請確認解壓縮後，可以在linux server上成功編譯並且執行。
:::

## Demo
*    時間: 未定
*    地點: 未定

:::info
在Linux中運行程式，詳細的呈現每一項功能，沒有demo的組別期末專題0分
:::

## Thanks for the following open source projects
*   CImg (https://github.com/GreycLab/CImg)
*   libjpeg (https://github.com/kornelski/libjpeg)
*   catimg (https://github.com/posva/catimg)
*   Photo Mosaic(https://github.com/coherent17/NYCU-OOP-Final-Project)
        
## 最終成果(github link)
