Bildiğimiz gibi _C++_ dili sanal işlevler yoluyla çok biçimliliğe güçlü bir destek veriyor. Sanal işlevlerin kullanılması durumunda hangi işlevin çağrıldığı programın çalışma zamanında anlaşıldığından bu tür çok biçimliliğe “dinamik çok biçimlilik” ya da _“çalışma zamanı çok biçimliliği”_ deniyor. Dinamik çok biçimliliğin getirdiği ek maliyetler var. Sanal işlevlere yapılan çağrılarda derleyici tipik olarak şöyle bir kod üretiyor:

* Sınıf nesnesinin içine bir gösterici veri elemanı gömülüyor. Sanal fonksiyona bir çağrı yapıldığında bu göstericiden sanal işlev tablosu deneilen bir veri yapısının adresi elde ediliyor.
* Bu veri yapısında derleme zamanında elde edilen bir indeks ile çağrılacak işlevin adresi elde ediliyor. Böyle bir kod da iki kez içerik alma _(dereferencing)_ maliyeti içeriyor.

Bellek kullanımı açısından baktığımızda da her sınıf nesnesinde bir gösterici için ilave bir bellek alanı kullanılıyor. Ayrıca türetme hiyerarşisi içindeki her sınıf için sanal işlev tablosu olarak kullanılan veri yapısının konumlandırılacağı bellek alanı gerekiyor. Özellikle küçük veriler taşıyan küçük sınıf nesneleri için bu bellek maliyeti bazen istenmiyor. Performans kritik uygulamalarda sanal işlevlerin faydalarını bir ölçüde bize sağlayacak ancak maliyeti azaltacak bazı başka çözümler var. İşte __CRTP__ bunu sağlayan örüntülerden _(pattern)_ biri.

__CRTP__, _C++_ dilinde sık kullanılan örüntülerden biri. Bu teknik 1980’li yıllarda  _"F-bounded quantification"_ olarak isimlendirilmiş. _1995_ yılında _Jim Coplien_ örüntüye bu ismi vermiş. O tarihten bugüne örüntü bu isimle biliniyor. Türkçeye kelime kelime çevirirsek _"meraklıca yinelenen şablon örüntüsü"_ diyebiliriz. _(demesek daha iyi olur gibi geliyor.)_ Şimdi örüntümüze bakalım. Taban sınıf olarak kullanılacak bir sınıf şablonumuz var ve türemiş sınıfların her biri bu şablon sınıfın türemiş sınıf açılımından türetiliyor:

```
#include <iostream>

template <typename Derived>
struct Base
{
	void interface()
	{
		static_cast<Derived *>(this)->implementation();
	}
	
};

struct Der : Base<Der>
{
	void implementation()
	{
		std::cout << "Der implementasyonu\n";
	}
};

int main()
{
	Der myder;
	myder.interface();  // "Der implementasyonu"

}
```
İlginç bir yapı değil mi? _Der_ sınıfını bir sınıf şablonu olan _Base_'in, _Base<Der>_ açılımından kalıtım yoluyla elde ediyoruz. Dilin kurallarını çiğneyen hiç bir durum yok. Yukarıdaki örnekte şu nokta özellikle dikkatimizi çekmeli:

```
void Base<Der>::implementation()
```

fonksiyonu _Base_ sınıfından türetilen _Der_ sınıfının bildiriminden önce (yani derleyicinin bu sınıfı bilmesinden önce) bildirilmesine karşın işlevin kodu _Der_ sınıfının bilinmesinden sonra yazılacak.

Dikkatimizi çekmesi gereken ikinci bir nokta da taban sınıfın _interface_ üye fonksiyonu içinde yapılan aşağıdaki dönüşüm:

```
static_cast<Derived *>(this)
```

Dilin kurallarına göre bu dönüşümün geçerli olabilmesi için _Base_ ve _Derived_ sınıflarının aynı hiyerarşi içinde bulunması gerekiyor. Bu dönüşümle _Base_ sınıfı türünden nesnenin adresini _(Derived *)_ türüne dönüştürüyor ve tür dönüştürme operatöründen elde edilen adresle türemiş sınıfının _implementation_ isimli fonksiyonunu çağırıyoruz. Derleyici bu durumda _implementation_ ismini türemiş sınıfın isim alanında arıyor.

Bu teknik dinamik çok biçimliliğin maliyetinden kaçınarak ve çoğu zaman ilave esneklik kazandırarak sanal işlevlerin etkilerine benzer bir yapı oluşturuyor. __CRTP__'nin bu özel kullanımı bazı kişiler tarafından _"statik çok biçimlilik"_ ya da _"simüle edilmiş dinamik bağlama"_ olarak isimlendiriliyor. _Windows_'un _ATL_ ve _WTL_ kütüphanelerinde bu teknik yoğun olarak kullanılıyor. Tekniği anlamak için, sanal fonksiyonları olmayan bir sınıfı kafanızda canlandırın. Taban sınıf başka fonksiyonları kendi üye fonksiyonları yoluyla çağırıyor. Taban sınıftan bir türetme yaptığımızda taban sınıfın tüm veri elemanlarını ve üye fonksiyonlarını kalıtım yoluyla almış oluyoruz.

Şimdi bir örnek üzerinde dinamik çok biçimliliği statik çok biçimliliğe dönüştürelim. Aşağıdaki gibi bir dinamik çok biçimliliğe sahip sınıf hiyerarşimiz olsun:

```
#include <iostream>
#include <string>

class Pet
{
public:
	void makeSound()
	{
		std::cout << getSound() << "\n";
	}

	virtual std::string getSound() const = 0;
};

class Dog : public Pet
{
public:
	virtual std::string getSound() const override
	{
		return{ "hav hav" };
	}
};

class Cat : public Pet
{
public:
	virtual std::string getSound() const override
	{
		return{ "miyav miyav" };
	}
};

void petFunc(Pet &r)
{
	r.makeSound();
}

int main()
{
	Cat mycat;
	Dog mydog;

	mycat.makeSound();
	mydog.makeSound();
}
```
 
Şimdi bu yapıyı __CRTP__ örüntüsüne dönüştürelim:

```
#include <iostream>
#include <string>

template <typename T>
class Pet
{
	const T& thisObject() { return *static_cast<const T*>(this); }

public:
	void makeSound()
	{
		std::cout << thisObject().getSound() << "\n";
	}

	// Sanal bir fonksiyon bildirilmiyor. Ancak bu sınıf şablonu T türüne ilişkin 
	// std::string T::getSound() const" imzalı bir fonksiyon bulunacağına göre yazılıyor.
};

class Dog : public Pet<Dog> // Şablon tür parametresine dikkat
{
public:
	std::string getSound() const
	{
		return{ "hav hav" };
	}
};

class Cat : public Pet<Cat>   //şablon tür parametresine dikkat
{
public:
	std::string getSound() const
	{
		return{ "miyav miyav" };
	}
};

template<typename T>
void gfunc(Pet<T> &r)
{
	r.makeSound();
}

int main()
{
	Cat mycat;
	Dog mydog;

	gfunc(mycat);
	gfunc(mydog);
}
```

_Pet_ sınıfının _private_ bölümünde tanımlanan

```
const T& thisObject() { return *static_cast<const T*>(this); }
```

fonksiyonu tür dönüştürme kodlarını daha kolay yazabilmemiz için oluşturduğumuz bir yardımcı yalnızca. _C++_'ın fonksiyon şablonlarına ilişkin önemli bir kuralını da hatırlamanın tam zamanı. Bir fonksiyon çağrısı yapılmadığı sürece derleyici bir fonksiyon şablonundan ya da sınıf şablonunun üye fonksiyonundan bir kod üretmeyecek. _getSound_ fonksiyonlarının kodunun üretilmesini tetikleyen _main_ fonksiyonu içinde yapılan çağrılar:

```
mycat.makeSound();
mydog.makeSound();
```

Şüphesiz iki yapının ayrı ayrı avantajları ve dezavantajları var. __CRTP__ kullandığımız yapıda ne taban sınıf nesneleri içinde gömülü bir gösterici ne de çalışma zamanında bellekte sınıflar için oluşturulmuş bir işlev tablosu tutuluyor. Fonksiyon çağrısı doğrudan yapılıyor. Yani iki ayrı içerik alma işlemi _(dereferencing)_ yok.
Ayrıca dinamik çok biçimlilikteki kurallardan farklı olarak taban sınıfın fonksiyonunun imzasıyla türemiş sınıfın fonksiyonunun imzasının aynı olması gerekmiyor.
Diğer taraftan taban sınıftan yapılan her bir türetme için hem taban sınıf için hem de türemiş sınıflar için farklı kodlar oluşturulduğundan statik çok biçimlilik sağlayan bu yapıda oluşturulan kod miktarı daha fazla olma eğiliminde. Hangi işlevin çağrıldığı programın çalışma zamanında değil de derleme zamanında belli olduğu için bu yapının dinamik çok biçimliliğe göre daha kısıtlı bir kullanımı var. Dinamik çok biçimlilikteki gibi katı bir tür kontrolü yok.

__CRTP__'nin kullanımı bu örneklerle sınırlı değil. Bu serinin diğer yazılarında __CRTP__'nin farklı amaçlarla kullanımını da inceleyeceğiz.
