## 2.1 main()をどう定義するか
### 2.1.1. mainメソッドの一例
main()の中身はシミュレーションの内容によって変わるが、ユーザーによって定義されなければならない。またGeant4はmain()を与えないが、一例をここで紹介する。
例2.1は最も簡素なmainメソッドの一例である。
```C++
Example 2.1.  Simplest example of main()
 #include "G4RunManager.hh"
 #include "G4UImanager.hh"

 #include "ExG4DetectorConstruction01.hh"
 #include "ExG4PhysicsList00.hh"
 #include "ExG4ActionInitialization01.hh"

 int main()
 {
   // construct the default run manager
   G4RunManager* runManager = new G4RunManager;

   // set mandatory initialization classes
   runManager->SetUserInitialization(new ExG4DetectorConstruction01);
   runManager->SetUserInitialization(new ExG4PhysicsList00);
   runManager->SetUserInitialization(new ExG4ActionInitialization01);

   // initialize G4 kernel
   runManager->Initialize();

   // get the pointer to the UI manager and set verbosities
   G4UImanager* UI = G4UImanager::GetUIpointer();
   UI->ApplyCommand("/run/verbose 1");
   UI->ApplyCommand("/event/verbose 1");
   UI->ApplyCommand("/tracking/verbose 1");

   // start a run
   int numberOfEvent = 3;
   runManager->BeamOn(numberOfEvent);

   // job termination
   delete runManager;
   return 0;
 }
```
mainメソッドは2つのツールキットクラスG4RunManagerとG4UImanagerと3つのクラスExG4DetectorConstruction01, ExG4PhisicsList00, ExG4ActionInitialization01を実装している。
以下のセッションではそれぞれのクラスについて説明する。

### 2.1.2 G4RunManager
初めにmain内でやらなければならないことはG4RunManagerのインスタンスを生成することである。これはmain関数内で明示的に生成しなければならない唯一のマネージャークラスである。
このクラスはプログラムの流れをコントロールし、動作中のイベントループを管理する。マルチスレッドに対応している場合、G4RunManagerの代わりにG4MTRunManagerクラスを使用するべきである。

G4RunManagerインスタンスが生成されると、ほかのマネージャークラスも生成される。これらはG4RunManagerとともにデストラクトされる。
ランマネージャーはユーザーの初期化クラスのメソッドを含めた初期化手続きも担っている。ユーザーはこれらのランマネージャーにビルド、ランに必要なすべての情報を与えなければならない。
与える情報はおおまかに以下のものが挙げられる。

1.  検出器をどのように設置するか。
2.  すべての粒子、すべての物理現象についてシミュレーションするか。
3.  一番初めの粒子はどのように与えられるか。
4.  その他必要事項

mainでは、以下の行
```C++
 runManager->SetUserInitialization(new ExG4DetectorConstruction01);
 runManager->SetUserInitialization(new ExG4PhysicsList00);
 runManager->SetUserInitialization(new ExG4ActionInitialization01);
```
でそれぞれ定義している。


ExG4DetectorConstruction01クラスはG4VUserDetectorConstructionクラスを継承した初期化クラスの一例である。このクラスでは、
1.	形状
2.	材質
3.	検知領域
4.	検知領域の読み出し法
についてユーザーが定義しなければならない。

同様に、ExG4PhysicsList01はG4VUserPhisicsListを継承したクラスで、
1.	シミュレーションで使う粒子の種類
2.	シミュレーションするすべての物理現象
3.	粒子のエネルギーのカット領域
について定義する。

ExG4ActionInitialization01クラスもまたG4VUserActionInitializationクラスを継承し、
1.  シミュレーションの際の読み出される、いわゆるユーザークラス
2.   必須の一次粒子を定義
を定義する。

以下の一文
```C++
  runManager->Initialize();
```
は検出器の構築、物理減少の生成、断面積の計算を設定しランする。最後のランマネージャーのメソッドは
```C++
  int numberOfEvent = 3;
  runManager->beamOn(numberOfEvent);
```
は3回のbeamOnイベントを1回のループで読み出す。詳しいことは3.4.4節、または3.4節に載っている。
上で述べたのように、他のマネージャークラスはランマネージャーが生成されたときに自動で生成される。
他のマネージャーの一例は、 UIマネージャーであるG4UIManagerである。
mainのなかでUIマネージャーへのポインタを取得するには、次のようにする。
```C++
  G4UImanager* UI = G4UImanager::getUIpointer();
```
現在のプログラム例ではapplyCommandメソッドがプログラムへランの情報を表示させるように
三回呼び出されている。
ユーザーがシミュレーションを詳細にコントロールできるような幅広いコマンドが利用可能である。

### 2.1.3 User Initialization and Action Classes
#### 2.1.3.1 User classes
ユーザークラスには二種類あり、それらはユーザー初期化クラスとユーザーアクションクラスである。
ユーザー初期化クラスは初期化の段階で使用され、ラン中はユーザーアクションクラスが使用される。
ユーザー初期化クラスはG4RunManagerへsetUserInitializationメソッドを通して直接登録しなければならない。
一方ユーザーアクションクラスはG4UserActionInitializationクラスで定義されなければならない。

#### 2.1.3.2 User Interface Classes
3つのユーザー初期化クラスが必須である。それらはgeant4が提供する抽象クラスを継承しなければならない。
```C++
  G4VUserDetectorConstruction
  G4VUserPhysicsList
  G4VUserActionInitialization
```
geant4はこれらのデフォルトの振る舞いは提供しない。G4RunManagerはこれらの必須のクラスである
Initialize()とbeamOn()が呼び出されているかチェックする。
前節で言及したように、G4VUserDetectorConstructionクラスはユーザーに検出器を定義するよう要求し、
G4UserPhysicsListはユーザーに物理現象を定義するよう要求する。
検出器の定義は2.2節、2.3節で説明する。物理現象の定義は2.4節、2.5節で説明する。
ユーザーアクションクラスであるG4VUserPrimarygeneratorActionクラスは初期イベント状態を定義するよう
要求する。初期イベント生成は2.8節で説明する。

G4VUserActionInitializationクラスは少なくとも１つの必須なユーザーアクションであるG4VUserPrimaryGeneratorActionクラス
を含まなければならない。すべてのアクションクラスは次節で詳細に説明する。
```C++
Example 2.2.  Simplest example of ExG4ActionInitialization01
 #include "ExG4ActionInitialization01.hh"
 #include "ExG4PrimaryGeneratorAction01.hh"
 void ExG4ActionInitialization01::Build() const
 {
   SetUserAction(new ExG4PrimaryGeneratorAction01);
 }
```
#### 2.1.3.3  User Action Classes
 G4VUserPrimaryGeneratorActionクラスはユーザーが提供しなければならない必須のクラスである。
 それは一次粒子生成のインスタンスを作る。ExG4PrimarygeneratorAction01はG4VUserPrimaryGeneratorActionクラス
 を継承するユーザーアクションクラスの一例である。
 このクラスでユーザーは初期イベントの初期状態を詳述しなければならない。
 今クラスはパブリックな仮想関数、それぞれのイベントの最初に呼び出されるGeneratePrimaries()を持つ。
 詳細は2.6節で紹介する。geant4は初期イベントの生成のためのデフォルトの振る舞いを提供しないことに注意。
 geant4は次の5つのユーザーフッククラスを提供する。
1.  G4UserRunAction
2.  G4UserEventAction
3.  G4UserStackingAction
4.  G4userTrackingAction
5.  G4UserStoppingAction
これらの追加のユーザーアクションクラスは、いくつかの仮想関数をもっている。それらはシミュレーションの全レベルの追加手順の詳細を提供する。

### 2.1.4 G4UIManager and UI CommandSubmission
geant4はインターコンと呼ばれるカテゴリーを提供する。G4UIManagerはこのカテゴリーのマネージャークラスである。
このカテゴリーの機能を使用して、ユーザーの知らないポインタのオブジェクトクラスのメソッド設定を呼び出すことができる。
例2.3では様々なgeant4マネージャーの詳細度が設定されている。詳細な機能説明とインターコムの使い方は次章で説明する。
```C++
Example 2.3.  An example of main() using interactive terminal and visualization. Code modified from the previous example are shown in blue.

 #include "G4RunManager.hh"
 #include "G4UImanager.hh"

#ifdef G4UI_USE
 #include "G4VisExecutive.hh"
#endif


 #include "ExG4DetectorConstruction01.hh"
 #include "ExG4PhysicsList00.hh"
 #include "ExG4PrimaryGeneratorAction01.hh"

 int main()
 {
   // construct the default run manager
   G4RunManager* runManager = new G4RunManager;

   // set mandatory initialization classes
   runManager->SetUserInitialization(new ExG4DetectorConstruction01);
   runManager->SetUserInitialization(new ExG4PhysicsList00);

   // set mandatory user action class
   runManager->SetUserAction(new ExG4PrimaryGeneratorAction01);

   // initialize G4 kernel
   runManager->Initialize();

  // Get the pointer to the User Interface manager
  G4UImanager* UImanager = G4UImanager::GetUIpointer();


  if ( argc == 1 ) {
    // interactive mode : define UI session
    #ifdef G4UI_USE
    G4UIExecutive* ui = new G4UIExecutive(argc, argv);
    UImanager->ApplyCommand("/control/execute init.mac");
    ui->SessionStart();
    delete ui;
    #endif
  }
  else {
    // batch mode
    G4String command = "/control/execute ";
    G4String fileName = argv[1];
    UImanager->ApplyCommand(command+fileName);
  }


   // job termination
   delete runManager;
   return 0;
 }
```
 上の例ではまだ含まれていないが、アウトプットストリームも必要である。
 G4cout、G4cerrはgeant4で提供されるiostreamである。
 その使い方は標準の使い方とG4UIManagerによって操作されることを除き、全く同じである。
 このように文字の出力は他のウィンドウやファイルに出力されるかもしれない。
 これらのアウトプットストリームの操作は7.2.4節で詳述する。これらのオブジェクトは通常の
 cout, cerrの代わりに使われる。
