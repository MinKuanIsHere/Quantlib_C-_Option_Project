# 金融計算程式運用（二）期末報告


學生: 金融所 碩二 陳旻寬 111352021


## 摘要
- 參照Quantlib當中的選擇權定價模型
`.\Myproj\external\QuantLib-1.32\Examples\EquityOption\EquityOption.cpp`
- 觀察Quantlib如何使用Black-Scholes for European、Finite differences、Binomial Cox-Ross-Rubinstein等三種歐式選擇權定價方法。

## Start C++ and Quantlib
- 使用window 11中的Linux & Ubuntu運行C++和Quantlib
- Install Linux on windows 11
    - https://learn.microsoft.com/zh-tw/windows/wsl/install
    - 安裝使用單一命令執行 WSL 所需的所有項目。 以滑鼠右鍵按一下並選取 [以系統管理員身分執行]，在系統管理員模式中開啟 PowerShell 或 Windows 命令提示字元，輸入 `wsl --install` 命令，然後重新開機電腦。
    - 重開機後打開檔案總管是否有**Linux(企鵝圖示)** ![image](https://hackmd.io/_uploads/B1OJe0LKa.png)

    - 重開機後可以在**windows圖示**中輸入`Ubunto`，並打開。初始運行方式請參照同個網站說明。
- Install C++ and Quantlib on Linux
    - https://rpubs.com/zhenghuii/CppFin1
    - follow section 補充：在 Windows 電腦安裝 WSL（Linux），並使用 VS code 開發 C++
    - Boost是運行Quantlib必要程式`sudo apt-get install libboost-all-dev`， (https://www.quantlib.org/install/linux.shtml)
    - 記得安裝 Visual Studio Code 必要的『延伸模組』：如與 cmake相關、WSL 相關等

- 執行說明
    - `CMakeLists.txt`遵從講義CppFin1進行Build。
    - 在資料夾`src`中執行`main.cpp`，內容為使用Quantlib計算選擇權價格，包含Black-Scholes for European、Finite differences、Binomial Cox-Ross-Rubinstein等三種歐式選擇權定價方法。
    - 同時，也可以執行資料夾`bin`中的`MyApp`來運行。


## 引入庫和命名空間：
```
/* -*- mode: c++; tab-width: 4; indent-tabs-mode: nil; c-basic-offset: 4 -*- */ //標記版本，讓使用者可以得知當前程式碼所適用的版本
#include <ql/qldefines.hpp> 
#if !defined(BOOST_ALL_NO_LIB) && defined(BOOST_MSVC)
#  include <ql/auto_link.hpp>
#endif
#include <ql/instruments/vanillaoption.hpp>
#include <ql/pricingengines/vanilla/analyticeuropeanengine.hpp>
#include <ql/pricingengines/vanilla/binomialengine.hpp>
#include <ql/pricingengines/vanilla/fdblackscholesvanillaengine.hpp>
#include <ql/pricingengines/vanilla/integralengine.hpp>
#include <ql/pricingengines/vanilla/mceuropeanengine.hpp>
#include <ql/time/calendars/target.hpp>
#include <ql/utilities/dataformatters.hpp>
#include <iostream>
#include <iomanip>

using namespace QuantLib;
```
- `/* -*- mode: c++; tab-width: 4; indent-tabs-mode: nil; c-basic-offset: 4 -*- */`
    - 這是一個特殊的註釋，用於告訴支持Emacs或Vim這樣的文本編輯器關於文件的格式化設置。mode: c++ 表示文件應該被當作C++源代碼處理`tab-width: 4` 設置制表符寬度為4個空格。`indent-tabs-mode: nil` 表示不使用制表符進行縮進，而是使用空格。`c-basic-offset: 4` 設置基本縮進級別為4個空格。
- `#include <ql/qldefines.hpp>`
    - 引入了QuantLib庫的一些基本定義。qldefines.hpp 是QuantLib庫中的一個頭文件，它包含了庫的基本配置和定義。這是使用QuantLib進行金融計算時必須的一步。
- `#if !defined(BOOST_ALL_NO_LIB) && defined(BOOST_MSVC) #  include <ql/auto_link.hpp>
#endif `
    - 這些行使用條件編譯指令來決定是否包含 `auto_link.hpp`。
`#if !defined(BOOST_ALL_NO_LIB) && defined(BOOST_MSVC)` 檢查是否定義了 `BOOST_MSVC` 且沒有定義 `BOOST_ALL_NO_LIB`。這通常與使用Microsoft Visual C++編譯器和Boost庫有關。
`# include <ql/auto_link.hpp>` 如果條件為真，則包含 `auto_link.hpp`。這個頭文件用於自動鏈接到正確的QuantLib庫版本，特別是在使用Microsoft Visual C++時。
`#endif` 結束條件編譯塊。
- `#include <ql/xxx/xxx.hpp>` 為來自QuantLib的標頭檔，用於引入QuantLib庫。
- `using namespace QuantLib`; 允許我們直接使用QuantLib命名空間中的類和函數，而不需要每次都加上 `QuantLib::` 前綴。在Quantlib中`.cpp`或是`.hpp`都是用`namespace QuantLib{...}`開頭，所以在`main.cpp`就引用可以省去很多命名的麻煩。
- `#include <ql/instruments/vanillaoption.hpp>`當中有個關鍵是用來判斷選擇權的樣態，例如是歐式或美式，甚至是有無包含股利:
```
switch (exercise_->type()) {
    case Exercise::European:
        if (dividends.empty())
            engine = std::make_unique<AnalyticEuropeanEngine>(newProcess);
        else
            engine = std::make_unique<AnalyticDividendEuropeanEngine>(newProcess, dividends);
        break;
    case Exercise::American:
    case Exercise::Bermudan: 
```
- 當中使用switch我認為不是非常洽當，應該可以單純使用`ifelse`就可以解決，同時讓程式碼更加易讀。例如
```
if (Exercise==European){
    if (dividends.empty())
        engine = std::make_unique<AnalyticEuropeanEngine>(newProcess);
    else
        engine = std::make_unique<AnalyticDividendEuropeanEngine>(newProcess, dividends);
    break;
    }
else if {Exercise==American};
else if {Exercise==Bermudan};

```
- `#include <ql/pricingengines/vanilla/analyticeuropeanengine.hpp>`為進行選擇權定價的參數初始化。

- `#include <ql/pricingengines/vanilla/binomialengine.hpp>`為使用二元樹進行選擇權定價:
```
Real delta = (p1u - p1d) / (s1u - s1d);
```
- `Real` 在QuantLib中是一個自定義的數據類型。它通常被定義為 `double`，但這種定義可以根據需要更改。這意味著QuantLib可以選擇使用不同的數據類型來定義 `Real`，而不必修改庫的每個部分。`double` 是C++標準中定義的原生數據類型。它通常用於表示雙精度浮點數，並在大多數平台上遵循IEEE 754標準。
- `#include <ql/time/calendars/target.hpp>`和
`#include <ql/utilities/dataformatters.hpp>`都是在Quantlib中使用日期格式的統一標準，方便使用。

## 主函數（main）的設置

```
 try {...
     // Here is option pricing code
     ...
     return 0;
     } catch (std::exception& e) {
        std::cerr << e.what() << std::endl;
        return 1;
    } catch (...) {
        std::cerr << "unknown error" << std::endl;
        return 1;
    }
```
- `try` 和 `catch` 塊用於異常處理。如果在 `try` 塊中的代碼執行過程中出現錯誤，則會跳轉到 `catch` 塊，並打印錯誤信息。 `catch` 塊提供了一種機制來優雅地處理程序中可能發生的預期和非預期異常。這樣做的好處是，即使程序遇到錯誤，它也不會突然崩潰，而是會提供一些錯誤信息，並以一種受控的方式結束。這對於調試和維護大型軟件應用來說非常重要。
- `try`執行的程式為我們想要執行的選擇權定價。
- `catch (std::exception& e)` 用於捕獲派生自 `std::exception` 的異常。`std::exception` 是C++標準庫中所有標準異常類的父類別(基類)。當捕獲到這樣的異常時，會執行以下操作：
    - `std::cerr << e.what() << std::endl;`：這行代碼將異常對象 `e` 的描述信息輸出到標準錯誤流（`std::cerr`）。`e.what()` 是 `std::exception` 類中定義的一個成員函數，它返回一個表示異常詳細信息的字符串。
    - `return 1;`：這表示程序以錯誤代碼 1 結束。在操作系統中，非零的退出代碼通常表示程序遇到了錯誤。
- `catch (...)`
這個 `catch` 塊是一個“捕獲所有”類型的處理器，它可以捕獲所有類型的異常，包括那些不是派生自 `std::exception` 的異常。當捕獲到這樣的異常時，會執行以下操作：
    - `std::cerr << "unknown error" << std::endl;`：由於這類異常可能不包含具體的錯誤信息，因此這行代碼只是將一個通用的錯誤消息輸出到標準錯誤流。
    - `return 1;`：同樣表示程序以錯誤代碼 1 結束。
    
## 參數設定

### 日期參數設定
```
//日期參數設定
Calendar calendar = TARGET(); //獲取歐洲交易的日曆
Date todaysDate(10, January, 2024); //起始日
Date settlementDate(13, January, 2024); //到期日
Settings::instance().evaluationDate() = todaysDate; //當前日期
```
- `Calendar calendar = TARGET();`
    - 創建一個 `Calendar` 對象，並將其初始化為 `TARGET` 日曆。
    - TARGET（Trans-European Automated Real-time Gross settlement Express Transfer system）是歐洲的主要支付系統，代表了歐元區的標準交易日曆。TARGET 日曆提供了一個常用的、標準化的交易日曆，適用於許多涉及歐元區的金融計算。
    - `Calendar` 是一個專門用於表示和處理日曆的類型，在QuantLib中用於處理與日期相關的各種計算。
- `Date todaysDate(10, January, 2024);`
    - 創建一個 `Date` 對象，表示的日期是2024年1月10日。設定“今天的日期”是許多金融計算的起點，例如，在定價選擇權或計算利息時。
    - `Date` 是QuantLib中用於表示日期的類型，提供一種標準化的方式來處理日期。
- `Date settlementDate(13, January, 2024);`
    - 創建一個 Date 對象，名為 settlementDate，指結算日期，此處設定日期是2024年1月13日。
- `Settings::instance().evaluationDate() = todaysDate;`
    - 將QuantLib的全局設置中的評估日期設置為 `todaysDate`。
    - 設置全局評估日期是為了確保所有的定價和計算都是基於同一個“當前日期”。
    - 使用 `Date` 類型作為 `evaluationDate` 的值，以保持數據類型的一致性，並利用 `Date` 類型提供的功能。

### 選擇權參數設定
```
//選擇權參數設定
Option::Type type(Option::Put);
Real underlying = 17465; //現貨價格
Real strike = 17500; //履約價
Spread dividendYield = 0.00; //股利率
Rate riskFreeRate = 0.01; //無風險利率
Volatility volatility = 0.03; //波動率
Date maturity(17, January, 2024);  //合約存續
DayCounter dayCounter = Actual365Fixed();
```

- `Option::Type type(Option::Put);`
    - 定義選擇權的類型為賣權（Put）。
    - Option::Type 是一個枚舉型態，用於清晰地表示選擇權的類型。
    - 若想要使用 Call (買權)則設定為`Option::Type type(Option::Call);`。
    - 在路徑`\external\QuantLib-1.32\ql\option.hpp`中可以看到更加詳細的設定
```
class Option : public Instrument {
public:
class arguments;
enum Type { Put = -1,
            Call = 1
};
Option(ext::shared_ptr<Payoff> payoff, ext::shared_ptr<Exercise> exercise)
: payoff_(std::move(payoff)), exercise_(std::move(exercise)) {}
void setupArguments(PricingEngine::arguments*) const override;
ext::shared_ptr<Payoff> payoff() const { return payoff_; }
ext::shared_ptr<Exercise> exercise() const { return exercise_; };
protected:
// arguments
ext::shared_ptr<Payoff> payoff_;
ext::shared_ptr<Exercise> exercise_;
};
```
- `\external\QuantLib-1.32\ql\option.hpp`說明
    - 枚舉 `Type`：`enum Type { Put = -1, Call = 1 };` 定義了一個名為 `Type` 的枚舉，其中包含兩個可能的值：Put 和 Call。這些值分別用 -1 和 1 表示。
    - 類別`class`成員：`ext::shared_ptr<Payoff> payoff_;` 和`ext::shared_ptr<Exercise> exercise_;` 是 Option 類別的成員，分別表示選擇權的損益和行使方式。
    - 構造函數：`Option(ext::shared_ptr<Payoff> payoff, ext::shared_ptr<Exercise> exercise)` 是 Option 類別的構造函數，用於創建 Option 對象。
    - 在 `Option::Type type(Option::Put);` 時，實際上是在做以下幾件事情：
        - 使用 `Type` 枚舉：您引用了在 Option 類別中定義的 `Type` 枚舉。
        - 創建 `type` 變數：您創建了一個名為 `type` 的變數，其型別為 `Option::Type`。
        - 初始化 `type` 變數：您將 `type` 變數初始化為 `Option::Put`，這意味著 `type` 變數現在表示一個賣權。 
    
- `Real underlying = 17465;`
    - 設定標的資產的當前市場價格(現貨價格)。
    - `Real` 通常是一個浮點數型態，用於表示有小數部分的數值，適合用於表示價格。
- `Real strike = 17500;` 
    - 設定選擇權的履約價格。
    - 使用 Real 型態可以確保履約價的精確表示。 
- `Spread dividendYield = 0.00;`
    - 設定標的資產的股利率。
    - `Spread` 用於表示利差或類似的金融數值，適合用於表示股利率。
- `Rate riskFreeRate = 0.01;`
    - 設定無風險利率。
    - `Rate` 是專門用於表示利率的數據型態，有助於在代碼中清晰地表達這是一個利率值。
- `Volatility volatility = 0.03;`
    - 作用：設定標的資產價格的波動率。
    - `Volatility` 型態專門用於表示波動率，這有助於在代碼中明確區分不同的數值類型。
- `Date maturity(17, January, 2024);`
    - 設定選擇權的到期日。
    - `Date` 型態用於表示日期。
- `DayCounter dayCounter = Actual365Fixed();`
    - 選擇使用 `Actual365Fixed` 日計數標準。
    - `DayCounter` 是一種用於計算時間間隔的類型，`Actual365Fixed` 是其中一種常用的標準，假設一年固定為365天。

### 參數設定的結果印出
```
//將參數設定print出
std::cout << "Option type = "  << type << std::endl;
std::cout << "Maturity = "        << maturity << std::endl;
std::cout << "Underlying price = "        << underlying << std::endl;
std::cout << "Strike = "                  << strike << std::endl;
std::cout << "Risk-free interest rate = " << io::rate(riskFreeRate)
          << std::endl;
std::cout << "Dividend yield = " << io::rate(dividendYield)
          << std::endl;
std::cout << "Volatility = " << io::volatility(volatility)
          << std::endl;
std::cout << std::endl;
std::string method;
```

- `std::cout << ... << std::endl;`
    - 這類程式碼都為print出結果
- `io::rate(riskFreeRate)`
    - 用於格式化並輸出無風險利率`(riskFreeRate)`的值。
    - `io::rate()` 將這個小數格式化為更易於閱讀和理解的百分比形式（例如將 0.01 格式化為 1%）。提高了輸出的可讀性，使得無風險利率的值更容易被用戶理解。
- io::rate(dividendYield)
    - 用於格式化並輸出股利率 ``(dividendYield)`` 的值。
    - 股利率也通常表示為小數，使用 `io::rate()` 可以將其轉換為更直觀的百分比形式。這樣做可以使得股利率的輸出更加清晰，幫助用戶快速理解這一重要參數。
- `io::volatility(volatility)`
    - 用於格式化並輸出波動率 `(volatility)` 的值。波動率是選擇權定價中的一個關鍵參數，清晰的輸出格式有助於更好地理解和評估這一參數。
    - 波動率同樣通常以小數形式表示，`io::volatility()` 將其格式化為百分比，使得輸出更加直觀。

## 建立選擇權定價模型的前期設定

### 設定輸出格式
```
Size widths[] = { 35, 14, 14, 14 };
std::cout << std::setw(widths[0]) << std::left << "Method"
          << std::setw(widths[1]) << std::left << "European"
          << std::endl;
```
- 設定輸出列的寬度，並輸出了兩個列標題："Method" 和 "European"。為了在輸出中創建整齊的列格式，方便閱讀和理解輸出的數據。
    - `Size` 是一種用於表示大小或數量的數據型態，這裡用於設定列寬。
    - `std::setw` 和 `std::left` 來自標準庫 `<iomanip>`，用於設定輸出格式。
    - 先忽略選擇權的計算過程，根據這樣的輸出設定，output為
    
```
Method                             European
Black-Scholes                      55.405512
Finite differences                 55.405903  
Binomial Cox-Ross-Rubinstein       55.401123
MC (crude)                         55.406298   
```

### 創建歐式選擇權定價模型

```
auto europeanExercise = ext::make_shared<EuropeanExercise>(maturity);

Handle<Quote> underlyingH(ext::make_shared<SimpleQuote>(underlying));

// bootstrap the yield/dividend/vol curves
Handle<YieldTermStructure> flatTermStructure(
    ext::make_shared<FlatForward>(settlementDate, riskFreeRate, dayCounter));
Handle<YieldTermStructure> flatDividendTS(
    ext::make_shared<FlatForward>(settlementDate, dividendYield, dayCounter));
Handle<BlackVolTermStructure> flatVolTS(
    ext::make_shared<BlackConstantVol>(settlementDate, calendar, volatility,
                             dayCounter));
auto payoff = ext::make_shared<PlainVanillaPayoff>(type, strike);
auto bsmProcess = ext::make_shared<BlackScholesMertonProcess>(
        underlyingH, flatDividendTS, flatTermStructure, flatVolTS);

// options
VanillaOption europeanOption(payoff, europeanExercise);
```

- `auto europeanExercise = ext::make_shared<EuropeanExercise>(maturity);`
    - 創建一個歐式選擇權的行使條件。
    - 使用智慧指標`（make_shared）`來創建一個歐式選擇權的行使條件對象。歐式選擇權只能在到期日行使。
    - `EuropeanExercise`在路徑`\Myproj\external\QuantLib-1.32\ql\exercise.hpp`，內容為：
```
class EuropeanExercise : public Exercise {
    public:
    EuropeanExercise(const Date& date);
};
```
- - `EuropeanExercise(const Date& date)`為呼叫實作，實作`EuropeanExercise`在路徑`\Myproj\external\QuantLib-1.32\ql\exercise.cpp`，內容為：
```
EuropeanExercise::EuropeanExercise(const Date& date)
: Exercise(European) {
    dates_ = std::vector<Date>(1,date);
}
```
- - 由於式歐式選擇權，所以執行日期只會在到期日，因此在`main.cpp`中的參數為`maturity`。此處只是確定到期日而已。
 

- `Handle<Quote> underlyingH(ext::make_shared<SimpleQuote>(underlying));`
    - 創建一個包含標的資產價格的引用。在定價模型中，需要引用標的資產的當前市場價格。
    - `Handle<Quote>` 是一種智慧指標，用於安全地引用價格數據。`SimpleQuote` 是一個存儲單一價格的簡單容器。
    - 這些函式庫作用都是QuantLib庫的一部分，用於處理金融數據和定價模型。
    - 優點：提供了一種靈活且安全的方式來引用和更新價格數據。
    - 缺點：相比直接使用原始數據，使用這些抽象可能稍微增加了複雜性。`
    - 使用裸指標(raw pointer)需要手動管理記憶體的生存期。意指需要開發者負責適時地釋放該記憶體，否則可能導致記憶體洩漏或者存取無效記憶體的問題。為了解決裸指標可能帶來的各種使用問題，C++ 引入智慧指標(smart pointer)，對裸指標進行包裝，使得智慧指標能夠『自動釋放所指向的物件。
- `Handle<YieldTermStructure> flatTermStructure(
    ext::make_shared<FlatForward>(settlementDate, riskFreeRate, dayCounter));`
    - 創建一個表示無風險利率結構的對象。
    - `YieldTermStructure` 是 QuantLib 中用於表示利率結構的類，它提供了未來不同時間點的利率。`FlatForward` 是一種特定的 `YieldTermStructure`，它表示一個恆定的、隨時間不變的利率。在這裡，它被用來表示一個恆定的無風險利率。
    - `Handle` 是一種智慧指標，用於管理 `YieldTermStructure` 對象的生命週期，並提供對這個對象的安全訪問。

- `Handle<YieldTermStructure> flatDividendTS(
    ext::make_shared<FlatForward>(settlementDate, dividendYield, dayCounter));`
    - 創建一個表示股利率結構的對象。
    - 使用 `FlatForward` 來表示一個恆定的股利率。這對於計算那些支付股利的標的資產（如股票）的選擇權價格非常重要。

- `Handle<BlackVolTermStructure> flatVolTS(
    ext::make_shared<BlackConstantVol>(settlementDate, calendar, volatility, dayCounter));`
    - 創建一個表示波動率結構的對象。
    - `BlackVolTermStructure` 是用於表示波動率結構的類，它提供了未來不同時間點的波動率。`BlackConstantVol` 是一種特定的 `BlackVolTermStructure`，用於表示一個恆定的波動率。在這裡，它被用來表示標的資產價格的恆定波動率。這對於使用 Black-Scholes-Merton 模型等定價模型來計算選擇權價格非常重要。
    
- `auto payoff = ext::make_shared<PlainVanillaPayoff>(type, strike);`
    - 創建一個表示選擇權損益方式的對象。
    - 在定價模型中，需要定義選擇權的支付方式，這裡使用的是普通香草選擇權（Plain Vanilla Option）的支付方式，即在到期時根據選擇權的類型（call或put）和履約價進行結算。
    - 使用 `auto` 和 `ext::make_shared` 可以簡化代碼並自動推斷出 payoff 的正確類型。`PlainVanillaPayoff` 是QuantLib中用於表示普通香草選擇權支付方式的類。
    - 實作路徑`\Myproj\external\QuantLib-1.32\ql\instruments\payoffs.cpp`內容為：
```
Real PlainVanillaPayoff::operator()(Real price) const {
    switch (type_) {
      case Option::Call:
        return std::max<Real>(price-strike_,0.0);
      case Option::Put:
        return std::max<Real>(strike_-price,0.0);
      default:
        QL_FAIL("unknown/illegal option type");
    }
}
```
- - 此處的payoff就歐式選擇權定義一樣，Call的payoff為$max(S-K, 0)$，Put的payoff為$max(K-S, 0)$。只是當中`price`為現貨價$S$，`strike`為履約價$K$。當然在`type`會根據先前的定義再決定這邊是要用哪個`case`的計算選擇權payoff。
    
- `auto bsmProcess = ext::make_shared<BlackScholesMertonProcess>(
    underlyingH, flatDividendTS, flatTermStructure, flatVolTS);`
    - 創建一個用於定價的Black-Scholes-Merton過程對象。
    - Black-Scholes-Merton過程是一種廣泛使用的選擇權定價模型，它考慮了標的資產的價格、無風險利率、股利率和波動率。
    - 使用 `auto` 和 `ext::make_shared` 提供了一種簡潔且安全的方式來創建和管理這個過程的實例。
    - 實作`\Myproj\external\QuantLib-1.32\ql\processes\blackscholesprocess.cpp`，內容為
```
BlackScholesMertonProcess::BlackScholesMertonProcess(
                          const Handle<Quote>& x0,
                          const Handle<YieldTermStructure>& dividendTS,
                          const Handle<YieldTermStructure>& riskFreeTS,
                          const Handle<BlackVolTermStructure>& blackVolTS,
                          const ext::shared_ptr<discretization>& d,
                          bool forceDiscretization)
```
- - 由`main.cpp`中可以發現只有用到前4個變數，目的為給予BS model初始化變數。分別為標的資產的價格、無風險利率、股利率和波動率。

    
- `VanillaOption europeanOption(payoff, europeanExercise);`
    - 創建一個歐式選擇權的實例。
    - 將之前定義的損益方式和選擇權行使方式結合起來，創建一個具體的選擇權實例。
    - `VanillaOption` 是QuantLib中用於表示普通香草選擇權的類。
    - 實作`\Myproj\external\QuantLib-1.32\ql\instruments\vanillaoption.cpp`內容為
```
VanillaOption::VanillaOption(
    const ext::shared_ptr<StrikedTypePayoff>& payoff,
    const ext::shared_ptr<Exercise>& exercise)
: OneAssetOption(payoff, exercise) {}
```
- - 實作`\Myproj\external\QuantLib-1.32\ql\instruments\oneassetoption.cpp`內容為
```
OneAssetOption::OneAssetOption(
    const ext::shared_ptr<Payoff>& payoff,
    const ext::shared_ptr<Exercise>& exercise)
: Option(payoff, exercise) {}
```
- - 兩個函數接收 payoff 和 europeanExercise 作為參數，並將它們傳遞給 VanillaOption 的基類 OneAssetOption 的構造函數。這樣，europeanOption 對象就根據提供的選擇權損益計算方式和履約方式被正確地構造和初始化。

## 正式建立選擇權定價模型
- 使用Quantlib當中函示庫，並用Black-Scholes for European、Finite differences、Binomial Cox-Ross-Rubinstein、Monte Carlo Method等四種歐式選擇權定價方法。

### Black-Scholes for European
```
// Black-Scholes for European
method = "Black-Scholes";
europeanOption.setPricingEngine(ext::make_shared<AnalyticEuropeanEngine>(bsmProcess));
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::endl;
```
- BS model：
    - `method = "Black-Scholes";`：設定一個字符串變量 method，用於表示當前使用的定價方法。
    - `europeanOption.setPricingEngine(...);`：為歐式選擇權設定定價引擎，這裡使用的是基於 Black-Scholes 模型的分析解引擎。
    - `std::cout << ... << europeanOption.NPV() << std::endl;`：輸出定價方法和計算出的選擇權凈現值（Net Present Value, NPV）。
    - 使用 `std::string` 來存儲定價方法的名稱，因為它是一種靈活且易於使用的文本存儲方式。
    - 使用 `auto` 來自動推斷 `AnalyticEuropeanEngine` 的類型，這樣可以簡化代碼並減少錯誤。
    - `AnalyticEuropeanEngine` 是 QuantLib 庫中的一個類，提供了基於 Black-Scholes 模型的歐式選擇權定價功能。
    - `ext::make_shared` 為創建智慧指標，用於管理記憶體。

- 實作`\Myproj\external\QuantLib-1.32\ql\instrument.cpp`
```
void Instrument::setPricingEngine(const ext::shared_ptr<PricingEngine>& e) {
    if (engine_ != nullptr)
        unregisterWith(engine_);
    engine_ = e;
    if (engine_ != nullptr)
        registerWith(engine_);
    // trigger (lazy) recalculation and notify observers
    update();
}
```
- - 當中還會用到`\Myproj\external\QuantLib-1.32\ql\pricingengines\blackcalculator.cpp`去計算BS下的選擇權定價。

- - 實作`europeanOption.NPV()`
```
Instrument::Instrument()
: NPV_(Null<Real>()), errorEstimate_(Null<Real>()) {
    // this makes sense in general (if the evaluation date
    // changes, you probably want to recalculate) and can also
    // help avoid some edge cases when lazy objects only forward
    // their first notification.
    registerWith(Settings::instance().evaluationDate());
}
```

### Finite differences
```
// Finite differences
Size timeSteps = 801;
method = "Finite differences";
auto fdengine =
    ext::make_shared<FdBlackScholesVanillaEngine>(bsmProcess,
                                                  timeSteps,
                                                  timeSteps-1);
europeanOption.setPricingEngine(fdengine);
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::endl;
```
- 有限差分法
    - `Size timeSteps = 801;`：設定有限差分法中的時間步數。
    - `method = "Finite differences";`：指定當前使用的定價方法為有限差分法。
    - `auto fdengine = ...;`：創建一個基於有限差分法的定價引擎。
    - `europeanOption.setPricingEngine(fdengine);`：將此定價引擎應用於歐式選擇權。
    - `std::cout << ... << europeanOption.NPV() << std::endl;`：輸出定價方法和計算出的選擇權凈現值（Net Present Value, NPV）。
    - Size 通常是一個整數型態，用於指定數量或大小，這裡用於設定時間步數。
    - 使用 `auto` 可以自動推斷出 `FdBlackScholesVanillaEngine` 的類型，簡化代碼。
    - `FdBlackScholesVanillaEngine` 是 QuantLib 庫中的一個類，提供了基於有限差分法的選擇權定價功能。
    - `ext::make_shared` 用於創建智慧指標，這是一種現代C++中常用的記憶體管理方法。

### Binomial method: Cox-Ross-Rubinstein
```
// Binomial method: Cox-Ross-Rubinstein
method = "Binomial Cox-Ross-Rubinstein";
auto crrEngine = ext::make_shared<BinomialVanillaEngine<CoxRossRubinstein>>(bsmProcess, timeSteps);
europeanOption.setPricingEngine(crrEngine);
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::endl;
```
- CRR 二項式模型
    - `method = "Binomial Cox-Ross-Rubinstein";`：設定一個字符串變量 method，用於表示當前使用的定價方法為 CRR 二項式模型。
    - `auto crrEngine = ...;`：創建一個基於 CRR 方法的二項式定價引擎。
    - `europeanOption.setPricingEngine(crrEngine);`：將此定價引擎應用於歐式選擇權。
    - `std::cout << ... << europeanOption.NPV() << std::endl;`：輸出定價方法和計算出的選擇權凈現值（Net Present Value, NPV）。
    - 使用 `auto` 可以自動推斷出 `BinomialVanillaEngine<CoxRossRubinstein>` 的類型，簡化代碼。
    - `BinomialVanillaEngine<CoxRossRubinstein>` 是 QuantLib 中用於實現 CRR 方法的類。
    - `BinomialVanillaEngine` 是 QuantLib 庫中的一個類，提供了基於二項式模型的選擇權定價功能。
    - `ext::make_shared` 用於創建智慧指標，這是一種現代C++中常用的記憶體管理方法。

### Monte Carlo Method: MC (crude)
```
// Monte Carlo Method: MC (crude)
timeSteps = 1;
method = "MC (crude)";
Size mcSeed = 42;
auto mcengine1 = MakeMCEuropeanEngine<PseudoRandom>(bsmProcess)
    .withSteps(timeSteps)
    .withAbsoluteTolerance(0.02)
    .withSeed(mcSeed);
europeanOption.setPricingEngine(mcengine1);
// Real errorEstimate = europeanOption.errorEstimate();
std::cout << std::setw(widths[0]) << std::left << method
          << std::fixed
          << std::setw(widths[1]) << std::left << europeanOption.NPV()
          << std::endl;
```
- 蒙地卡羅
    - `timeSteps = 1;`：設定蒙特卡羅模擬中的時間步數。
    - `method = "MC (crude)";`：指定當前使用的定價方法為粗略的蒙特卡羅模擬。
    - `auto mcengine1 = ...;`：創建一個基於蒙特卡羅方法的定價引擎。
    - `europeanOption.setPricingEngine(mcengine1);`：將此定價引擎應用於歐式選擇權。
    - `std::cout << ... << europeanOption.NPV() << std::endl;`：輸出定價方法和計算出的選擇權凈現值（Net Present Value, NPV）。
    - `Size` 用於表示數量，這裡用於設定隨機數種子。
    - 使用 `auto` 可以自動推斷出 `MakeMCEuropeanEngine` 的類型，簡化代碼。
    - `MakeMCEuropeanEngine` 是 QuantLib 庫中的一個函數，用於創建基於蒙特卡羅方法的選擇權定價引擎。
    - `PseudoRandom` 是一種偽隨機數生成器，用於蒙特卡羅模擬。
