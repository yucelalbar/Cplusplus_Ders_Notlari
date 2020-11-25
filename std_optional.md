# std::optional Sınıfı 

_C++17_ standartları ile standart kütüphanemizin bir eksiği daha tamamlandı. Artık bizim de bir _optional_ sınıfımız var. Bu yazımızda _std::optional_ sınıfını ayrıntılı olarak ele alacağız.

Programlamada sıklıkla karşımıza çıkan bir durum var: Bir koşul sağlandığında bir nesne oluşturup o nesneyi kullanmamız gerekiyor. Ama bu koşul sağlanmadığında ise bir nesneye ihtiyacımız kalmıyor dolayısıyla bir nesne oluşturmamız gerekmiyor. İşte _std::optional_ sınıfı böyle durumlarda kullanılıyor.

*std::optional<T>* türünden bir nesne, programın çalışma zamanının belirli bir noktasında *T* türünden bir değer tutuyor ya da tutmuyor durumda olabilir. _optional_ nesnesinin bir değere sahip olması kadar bir değere sahip olmaması da son derece doğal bir durum. _std_ isim alanı içinde tanımlanan _optional_ sınıf şablonunun bildirimi şöyle:

```
template<typename T>
class optional;
```

Burada T, _optional_ nesnesinin tutabileceği nesnenin türü.

Peki böyle bir sınıfın gerçekleştirimi nasıl yapılabilir dersiniz? _std::optional_ sınıfı türünden bir nesne aslında _T_ türünden bir nesne için kullanılacak bir bellek alanına ve bir de _bool_ değişkeni tutacak bellek alanına sahip. _bool_ türden değişken _std::optional_ nesnesinin _T_ türünden bir nesneye sahip olup olmadığını gösterecek bir bayrak olarak kullanılıyor.

Herhangi türden bir nesneyi _bool_ türden bir bayrak değişken ile birlikte bir sınıf oluşturacak şekilde sarmalayabiliriz. 
Sarmalanmış yapı içindeki bayrak elemanı, kullanıcı kodlara bir değerin tutulduğu ya da tutulmadığı konusunda bilgi verebilir. İngilizcede bu şekilde oluşturulmuş türlere _"nullable types"_ deniyor. Böyle bir türden bir nesnenin kullanıcıları, bir yorum satırına gerek kalmaksızın, nesnenin bir değer tutup tutmadığını sorgulayabilirler. Programlama dünyasında opsiyonel türler yeni değil. Örneğin _Haskel_ dilinde yer alan _Data.maybe_ opsiyonel türlerin en eskilerinden biri. 
_optional_ sınıfı _2003_ yılından bu yana _Boost_ kütüphanesinin kullanılan öğelerinden biri. 
Standart kütüphanenin _optional_ sınıfının tasarımında da _Boost_ kütüphanesinin deneyiminden büyük ölçüde faydalanılmış.

_std::optional_ bir değer türü _(value type)_ oluşturuyor. 
Yani kopyalama işlemleri ile aynı değere sahip farklı nesneler oluşturulabiliyor. 
Diğer taraftan bir _optional_ nesnesi tutacağı değer için kendisi için ayrılmış bir bellek alanınını kullanıyor. 
Yani dinamik bir bellek alanı kullanmıyor. 
Aşağıdaki kodu derleyin ve ekran çıktısını yorumlamaya çalışın:

```
#include <optional>
#include <iostream>

template<size_t n>
class A {
	unsigned char buffer[n];
};

template<size_t n>
using optype = std::optional<A<n>>;


int main()
{
	std::cout << sizeof(optype<128>) << '\n';
	std::cout << sizeof(optype<256>) << '\n';
	std::cout << sizeof(optype<512>) << '\n';
	std::cout << sizeof(optype<1024>) << '\n';
	std::cout << sizeof(optype<2048>) << '\n';
}
```

## _optional_ nesnelerinin oluşturulması

_optional_ nesnelerinin oluşturulması için birden fazla yol var. 
Bunlardan biri, bir nesne hayata getirmeyen yani değer tutmayan (boş) bir _optional_ nesnesi oluşturmak:

```
#include <optional>

int main()
{
	std::optional<int> op1;
	std::optional<int> op2{};
	std::optional<int> op3{std::nullopt };
	//...
}
```

Yukarıdaki kodda _optional<int>_ türünden *op1, op2* ve _op3_ isimli nesneler hayata boş olarak getiriliyorlar. 
_op3_ için çağrılan kurucu işleve argüman olarak *"std::nullopt"* ifadesinin gönderildiğini görüyorsunuz. 
*<optional>* başlık dosyasında *nullopt_t* isimli bir boş sınıf _(empty class)_ tanımlanmış. 
_nullopt_, bu boş sınıf türünden oluşturulan ve sabit ifadesi olarak kullanılabilen _constexpr_ bir sınıf nesnesi. 

```
#include <stdio.h>

int main()
{
	int x;

	printf("bir tamsayi girin: ");
	int retval = scanf("%d", &x); //ctrl Z

	printf("retval = %d\n", retval); 

}
```

`optional` sınıfının `nullopt_t` türünden kurucu işlevi, `nullopt` sabiti ile çağrıldığında bu kurucu işlev boş bir optional nesnesi hayata getiriyor. Yine bir `optional` değişkenine bu sabitin atanması `optional` nesnesinin sarmaladığı değişkenin hayatını sonlandırıyor, böylece` optional` nesnesi boşaltılmış oluyor:

```
#include <optional>

int main()
{
	std::optional<int> op{ 12 };
	//op nesnesi boş değil

	op = std::nullopt; 
	//op nesnesi boş durumda
}
```
Bir `optional` nesnesini dolu olarak da hayata getirebiliriz. Aşağıdaki koda bakalım:

```
#include <optional>
#include <string>

int main()
{
	using namespace std::literals;

	std::optional op1{ 34.5 };
	std::optional op2{"necati"}; //optional<const char *>
	std::optional op3{ "oguz"s }; //optional<string>
	//...
}
```

_op1, op2_, ve _op3_ nesnelerinin tanımında şablon tür argümanının kullanılmadığını görüyorsunuz.
Burada _C++17_ standartları ile dile eklenen ve popüler olarak _CTAD (constructor template argument deduction)_ diye isimlendirilen özellik kullanılıyor. 
Bu mekanizma ile derleyici sınıfın kurucu işlevine gönderilen argümanın türünden hareketle şablon tür argümanının çıkarımını yapabiliyor. 
Yukarıdaki kodda derleyici _op1_ nesnesine ilk değer veren ifadeden hareketle _op1_ nesnesinin türünün çıkarımını

```
std::optional<int>
```
olarak yapıyor. Benzer şekilde _op2_ nesnesinin türü için

```
std::optional<const char *>
```
_op3_ nesnesinin türü için de

```
std::optional<std::string>>
```
çıkarımları yapılıyor.
Şüphesiz _optional_ nesnesini oluştururken şablon tür argümanını istediğimiz gibi belirleyebiliriz:

```
#include <optional>
#include <string>
#include <complex>
#include <vector>

int main()
{
	std::optional<int> op1{ 12 };
	std::optional<std::string> op2{"irfan"};
	std::optional<std::complex<double>> op3{ std::complex{1., 2.} };
	std::optional<std::vector<int>> ivec{ {1, 3, 5, 7, 9} };
	//...
}
```

Ancak _optional_ nesnesinin kurucu işlevine birden fazla değer gönderilecek ise bu durumda şablon tür argümanı belirtilse dahi çıkarım yapılamıyor. 
Bu yüzden _optional_ sınıfının kurucu işlevine birden fazla argümanın gönderilmesi durumunda, tür çıkarımın yapılabilmesi için ilk argüman olarak *in_place* ifadesinin gönderilmesi gerekiyor. 
*std::in_place* standart *<utility>* başlık dosyasında tanımlanmış olan *in_place_t* isimli bir boş sınıf türünden _constexpr_ bir nesnenin ismi. 
Bu tür boş sınıfların ve boş sınıf nesnelerinin varlık nedeni derleyicinin çıkarım yapmasına olanak sağlamak. Aşağıdaki koda bakalım:

```
#include <optional>
#include <complex>
#include <set>
#include <cctype>

int main()
{
	//std::optional<std::complex<double>> op1{1, 2}; //gecersiz
	std::optional<std::complex<double>> op2{std::in_place, 1, 2 }; 
	auto f = [](char x, char y) {
		return std::toupper(x) > std::toupper(y);
	};
	std::optional<std::set<char, decltype(f)>> op3{ std::in_place, 
                                                    {'c', 'T', 'a', 'B'}, f }; 
}
```
## *make_optional* işlevi
_optional_ nesnelerini oluşturmanın bir başka yolu da _make_optional_ isimli global yardımcı işlevi çağırmak. 
Bu işleve birden fazla argüman geçsek de artık _in_place_ nesnesini işleve göndermek zorunda değiliz:

```
#include <optional>
#include <complex>

int main()
{
	auto op1 = std::make_optional(12); //optional<int>
	auto op2 = std::make_optional("neco"); //optional<const char *>
	auto op3 = std::make_optional<std::complex<double>>(1.2, 5.6); 
        //op3 nesnesinin turu : optional<complex<double>>
}
```

## optional nesnelerinin boş olup olmadığını sınamak
Bir _optional_ nesnesinin boş olup olmadığını yani bir değer tutup tutmadığını sınıfın _operator bool_ ya da _has_value_ isimli işlevleriyle sınayabiliriz:

```
constexpr explicit operator bool() const noexcept;
constexpr bool has_value()const noexcept;
```
Bir _optional_ nesnesini _nullopt_ değeriyle eşitlik/eşitsizlik karşılaştırmasına da sokabiliriz:

```
#include <optional>

std::optional<int> func();

int main()
{
	auto op = func();

	if (op) {/**/} //dolu ise
	if (op.has_value()) {/**/ } //dolu ise
	if (op != std::nullopt) {/**/ } //dolu ise

	if (!op) {/**/ } //bos ise
	if (!op.has_value()) {/**/ } //bos ise
	if (op == std::nullopt) {/**/ } //bos ise
	//...
}
```
## optional nesnesinin tuttuğu değere erişmek
_optional_ nesnesinin tuttuğu değere erişmenin yine birden fazla yolu var. 
Sınıfın içerik operatör ve ok operatör fonksiyonları ile tutulan nesneye ya da o nesnenin öğelerine erişebiliriz. 
Ancak bu operatörlerin operandı olan _optional_ nesnesinin boş olması durumunda tanımsız davranış _(undefined behavior)_ oluşuyor. 
Böyle bir erişimde bir hata nesnesi gönderilmiyor _(exception throw edilmiyor)_. Aşağıdaki koda bakalım:

```
#include <optional>
#include <string>
#include <iostream>

int main()
{
	std::optional<std::string> op{ "kaan" };
	std::cout << *op << "\n";
	*op += " aslan";
	std::cout << *op << "\n";
	std::cout << "uzunluk : " << op->size() << "\n";
	op = std::nullopt;
	std::cout << *op << "\n"; //tanımsız davranış
	//...
}
```
Yukarıdaki koddan da görüldüğü gibi operator `"*"` işlevi referans döndürüyor.

Tutulan nesneye güvenli bir şekilde erişim gerçekleştirmek için öncelikle _optional_ nesnesinin boş olmadığından emin olmalıyız:

```
#include <optional>
#include <iostream>

std::optional<int> func();

int main()
{
	if (auto op = func())
		std::cout << *op << "\n";

	//...
	if (auto op = func(); op)
		std::cout << *op << "\n";
	//...

	if (auto op = func(); op.has_value())
		std::cout << *op << "\n";

	if (auto op = func(); op.has_value())
		std::cout << *op << "\n";
	//...

}
```
## _value_ işlevi
Tutulan nesneye erişmenin bir başka yolu da sınıfın _value_ isimli üye işlevini çağırmak. 
_operator *_ işlevi gibi _value_ işlevi de tutulan nesneye referans döndürüyor. 
Boş bir _optional_ nesnesi için _value_ işlevinin çağrılması durumunda, _std::exception_ sınıfınından kalıtım yoluyla elde edilen _std::bad_optional_access_ türünden bir hata nesnesi gönderiliyor:

```
#include <optional>
#include <string>
#include <iostream>

int main()
{
	std::optional<std::string> op{ "oguz karan" };

	std::cout << op.value() << "\n";
	op.value().assign(5, 'A');
	std::cout << op.value() << "\n";
	op = std::nullopt;

	try {
		std::cout << op.value() << "\n";
	}
	catch (const std::bad_optional_access &ex) {
		std::cout << "hata yakalandi : " << ex.what();
	}
}
```

## *value_or* işlevi
Tutulan değere erişmenin bir başka yolu da sınıfın *value_or* isimli işlevini çağırmak. 
Bu işlev _value_ işlevinden farklı olarak bir argüman alıyor ve _optional_ nesnesinin boş olması durumunda kendisine gelen bu değeri döndürüyor:

```
#include <optional>
#include <string>
#include <iostream>

void display_e_mail(const std::optional<std::string> &op)
{
	std::cout << "e posta adresi : " << op.value_or("belirtilmemis") << "\n";
}

int main()
{
	std::optional<std::string> e_mail_address{ "necati@gmail.com" };
	display_e_mail(e_mail_address);
	e_mail_address = std::nullopt;
	display_e_mail(e_mail_address);
	//...	
}
```
_value_ işlevinden farklı olarak *value_or* işlevi referans döndürmüyor:

```
#include <optional>

int main()
{
	std::optional<int> op{ 10 };

	op.value() = 20;
	op.value_or(0) = 30; //gecersiz
	//...
}
```

## _optional_ nesnelerinin değerlerini değiştirmek
*std::optional<T>* sınıfı türünden bir nesnenin değerini değiştirmek için sınıfın atama operatörlerini kullanabiliriz. 
Atama operatörünün sağ terimi olan ifade

+ *optional<T>* türünden olabilir.
+ *optional<U>* türünden olabilir. (_U_ türünden _T_ türüne dönüşüm var ise)
+ _T_ türünden olabilir.
+ _U_ türünden olabilir. (_U_ türünden _T_ türüne dönüşüm var ise)
+ _std::nullopt_ değeri olabilir.
+ _{}_ ifadesi olabilir.

Aşağıdaki kodu inceleyelim:

```
#include <optional>
#include <iostream>


int main()
{
	std::optional<double> d1{ 2.56 };
	std::optional<double> d2{ 0.78};
	std::optional<int> i{ 40 };
	
	std::cout << *d1 << '\n';
	d1 = d2;
	std::cout << *d1 << '\n';
	d1 = i;
	std::cout << *d1 << '\n';
	d1 = 5.69;
	std::cout << *d1 << '\n';
	d1 = 13;
	std::cout << *d1 << '\n';
	d1 = std::nullopt;
	d2 = {};

	if (!d1 && !d2)
		std::cout << "her iki nesnenin de degeri yok\n";
}
```

## _emplace_ işlevi
_optional_ sınıfının en önemli işlevlerinden biri _emplace_. 
Bu işlev ile bir nesneyi kopyalama olmadan doğrudan _optional_ nesnesi içinde yer alan bellek alanında hayata başlatabiliyoruz. 
_emplace_ işlevi standart kütüphanedeki kap sınıflarının _emplace_ işlevlerinde olduğu gibi mükemmel gönderim _(perfect forwarding)_ mekanizmasından faydalanıyor. 
Dolu bir _optional_ nesnesi için _emplace_ işlevi çağrıldığında _optional_ nesnesi tutmakta olduğu nesnesin sonlandırıcı işlevini _(destructor)_ çağırıyor:

```
#include <iostream>
#include <optional>

class A {
public:
	~A() { std::cout << "~A() this: " << this << "\n"; }
	A() { std::cout << "A() this : " << this << "\n"; }
	A(int) { std::cout << "A(int) this : " << this << "\n"; }
	A(int, int) { std::cout << "A(int, int) this : " << this << "\n"; }
	A(double) { std::cout << "A(double) this : " << this << "\n"; }
	A(std::string) { std::cout << "A(string) this : " << this << "\n"; }
};

int main()
{
	std::optional<A> os;
	std::cout << "&os = " << &os << "\n";
	os.emplace();
	os.emplace(10);
	os.emplace(10, 20);
	os.emplace(4.5);
	os.emplace("necati");
}
```
Yukarıdaki kodu derleyip çalıştırın. 
_optional_ nesnesi dolu iken _emplace_ işlevi her çağrıldığında önce _A_ sınıfının sonlandırıcı işlevinin çağrıldığını daha sonra ise _A_ sınıfının uygun kurucu işlevinin çağrıldığını göreceksiniz.

## _optional_ nesneleri tarafından kontrol edilen nesnelerin ömürleri
Bir _optional_ nesnesinin hayatı bittiğinde _optional_ nesnesi dolu ise hayata getirilmiş nesnenin sonlandırıcı işlevi çağrılır. 
Ancak aşağıdaki durumlarda da _optional_ nesnesinin kontrol ettiği nesnenin sonlandırıcı işlevi çağrılır:

+ _optional_ nesnesinin _emplace_ isimli işlevinin çağrılması
+ _optional_ nesnesine _nullopt_ değerinin atanması
+ _optional_ nesnesine _{}_ ifadesinin atanması
+ _optional_ nesnesinin reset işlevinin çağrılması (sınıfın _reset_ isimli işlevinin çağrılmasıyla eğer _optional_ nesnesi boş değil ise kontrol edilen nesnenin ömrü sonlandırılır.)

```
#include <iostream>
#include <optional>

class A {
	int mx;
public:
	~A() { std::cout << "~A() mx = " << mx << "\n"; }
	A(int x) : mx{ x } {}
};

int main()
{
	std::optional<A> os(50);
	os.reset();
	os = 20;
	os = std::nullopt;
	os = 30;
	os = {};
	os = 40;
	os.emplace(50);
	os.emplace(60);
}
```
## _optional_ sınıfı ve taşıma semantiği
_optional_ sınıfı taşıma semantiğini de destekliyor. 
Bir _optional_ nesnesini başka bir _optional_ nesnesine taşıyabiliyoruz. 
Bu durumda içerilen bir nesne var ise o da taşınıyor. 
Aşağıdaki kodu derleyip çalıştırın:

```
#include <optional>
#include <iostream>

struct A {
	A() { std::cout << "A default ctor.()\n"; }
	A(const A &) = delete;
	A& operator=(const A &) = delete;
	A(A &&) { std::cout << "A move ctor()\n"; }
	//...
};

int main()
{
	std::optional<A> op1;
	op1.emplace(); //A default ctor.
	std::optional<A> op2{ std::move(op1) }; //A's move ctor.
	std::cout << std::boolalpha;
	std::cout << op1.has_value() << "\n";
	std::cout << op2.has_value() << "\n";
}
```
Yukarıdaki kodda, *op1* nesnesinin taşınması ile *op1*'in içerdiği *A* nesnesi taşınmış oluyor. 
Taşıma işleminden sonra *op1* nesnesi dolu olsa da içerdiği nesne taşınmış durumda *(moved-from state)*. 
İçerilen nesneye yeniden bir değer atamadan bu nesne yeniden kullanılmamalı.
İçerilecek nesneyi dışarıdan içeriye ya da içerilen nesneyi içeriden dışarıya taşımak da mümkün. 
Aşağıdaki koda bakalım:

```
int main()
{
	A ax;

	std::optional<A> op1{ std::move(ax) };
	std::optional<A> op2{ std::move(*op1) };
}
```
Yukarıdaki kodu derleyip çalıştırdığınızda `A` sınıfının taşıyan kurucu işlevinin `(move constructor)` iki kez çağrıldığını göreceksiniz.

## _optional_ nesneleri ve karşılaştırma işlemleri
*optional<T>* türünden bir nesne
a) *optional<T>* türünden bir nesne ile
b) *optional<U>* türünden bir nesne ile (eğer *T* ve *U* karşılaştırılabilir türler ise)
b) *T* türünden bir ifade ile
d) *U* türünden bir ifade ile (eğer *T* ve *U* karşılaştırılabilir türler ise)
c) *std::nullopt* değeri ile

karşılaştırılabilir. Karşılaştırılan değerler optional nesnelerinin tuttuğu değerlerdir.
Boş bir optional nesnesi değeri ne olursa olsun dolu bir optional nesnesinden daha küçük kabul edilir.
İki boş optional nesnesinin karşılaştırılmasından _true_ değeri elde edilir.
Aşağıdaki kodda yapılan karşılaştırma işlemlerini inceleyiniz:

```
#include <optional>
#include <iostream>

using namespace std;

int main()
{
	optional<int> oe;
	optional<int> ox{ 10 };
	optional<int> oy{ 20 };

	cout.setf(ios::boolalpha);

	cout << (oe == ox) << '\n'; //false
	cout << (oe == nullopt) << '\n'; //true
	cout << (oe < ox) << '\n'; //true
	cout << (ox > oy) << '\n'; //false
	cout << (ox == 10) << '\n'; //true

	optional<unsigned> oz;
	optional<unsigned> omin{ 0 };
	cout << (oz < omin) << '\n'; //true
}
```
*optional<bool>* nesneleri ile yapılan karşılaştırmalara özellikle dikkat edilmeli. Aşağıdaki koda da bakalım:

```
#include <optional>
#include <iostream>

using namespace std;

int main()
{
	optional<bool> oe{ nullopt };
	optional<bool> ox{ false };
	optional<bool> oy{ true};


	cout.setf(ios::boolalpha);

	cout << (oe == ox) << '\n';  //false
	cout << (oe == oy) << '\n';  //false
	cout << (oe < ox) << '\n';  //true
	cout << (oe < oy) << '\n';  //true
	cout << (oe == true) << '\n';  //false
	cout << (oe == false) << '\n';  //false
	cout << (ox == oy) << '\n';  //false
}
```
## optional sınıfının kullanıldığı tipik durumlar
+ Bir işlevin *optional* sınıfı türünden bir değer döndürmesi. 
Bazı işlevler bir koşula bağlı olarak bir değer döndürebilirler. 
Ancak koşul sağlanmadığında döndürecek değerleri olmayabilir. 
Yani işlevin bir değer döndürmesi kadar döndürmemesi de doğaldır. 
Bu tür durumlarda işlevin geri dönüş değeri _optional_ sınıfı türünden olabilir. 
Aşağıdaki kodu inceleyelim:

```
#include <optional>
#include <string>
#include <iostream>

std::optional<int> to_int(const std::string& s)
{
	try {
		return std::stoi(s);
	}
	catch (...) {
		return std::nullopt;
	}
}

int main()
{
	for (auto ptr : { "987", "007bond", "ali", "23haziran", "2312126" })
		if (auto op = to_int(ptr))
			std::cout << ptr << " icin tamsayi degeri: " << *op << "\n";
		else
			std::cout << ptr << " gecerli tamsayi icermiyor\n";
}
```
Yukarıdaki kodda tanımlanan *to_int* isimli işlev, bir *std::string* nesnesini bir tamsayıya dönüştürüyor.
Ancak işleve gönderilen yazının geçerli bir tamsayı ifade etmemesi durumunda işlevimizin geri döndüreceği bir tamsayı olamayacak. 
Bu yüzden işlevin geri dönüş değeri türünün

```
std::optional<int>
```
olarak seçildiğini görüyorsunuz. 
İşlev gelen yazıdan bir tamsayı elde edilemesi durumunda boş bir *optional<int>* nesnesi döndürüyor.
*to_int* isimli işlevi aşağıdaki gibi de tanımlayabilirdik:

```
std::optional<int> to_int(const std::string& s)
{
	std::optional<int> retval; 

	try {
		retval = std::stoi(s);
	}
	catch (...) {}
	
	return retval;
}
```

+ Bir işlevin parametre değişkeninin _optional_ sınıfı türünden olması. 
Bir işlevin bir parametresine, müşteri kodun bir değer göndermesi kadar değer göndermemesi de doğal bir durum ise işlevin parametre değişkeni _optional_ sınıfı türünden yapılabilir. 
Bu durumda işlevi çağıracak kod bu parametreye ya bir değer ya da _std::nullopt_ sabitini gönderebilir.

+ Bir sınıfın bir veri elemanının _optional_ sınıfı türünden olması.

Aşağıdaki kodda hem bir işlevin parametresinin hem de bir sınıfın bir veri öğesinin *std::optional* türünden olduğunu göreceksiniz:

```
#include <optional>
#include <iostream>
#include <string>

class Name
{
private:
	std::string m_first;
	std::optional<std::string> m_middle;
	std::string m_last;
public:
	Name(std::string first,
		std::optional<std::string> middle,
		std::string last)
		: m_first{ std::move(first) }, m_middle{ std::move(middle) }, 
		m_last{ std::move(last) } {
	}

	friend std::ostream& operator << (std::ostream& os, const Name& name) {
		os << name.m_first << ' ';
		if (name.m_middle) {
			os << *name.m_middle << ' ';
		}
		return os << name.m_last;
	}
};
```

Yukarıdaki kodda kişilerin isimlerini, temsil etmek amacıyla _Name_ isimli bir sınıfın tanımlandığını görüyorsunuz. 
Sınıfın *std::optional<std::string>* türünden *m_middle* isimli veri öğesi kişilerin sahip olabileceği ya da sahip olmayacağı ikinci isimlerini tutması için tanımlanmış.
