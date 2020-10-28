Bildiğimiz gibi C++ dili sanal işlevler yoluyla çok biçimliliğe güçlü bir destek veriyor. Sanal işlevlerin kullanılması durumunda hangi işlevin çağrıldığı programın çalışma zamanında anlaşıldığından bu tür çokbiçimliliğe “dinamik çokbiçimlilik” ya da “çalışma zamanı çok biçimliliği” deniyor. Dinamik çokbiçimliliğin getirdiği ek maliyetler var. Sanal işlevlere yapılan çağrılarda derleyici tipik olarak şöyle bir kod üretiyor:
i) Sınıf nesnesinin içine bir gösterici veri öğesi gömülüyor. Sanal işleve bir çağrı yapıldığında bu göstericiden sanal işlev tablosu deneilen bir veri yapısının adresi elde ediliyor.
ii) Bu veri yapısından bir indis kullanarak çağrılacak işlevin adresi elde ediliyor. Böyle bir kod da iki kez içerik alma (dereferencing) maliyeti içeriyor.

Bellek kullanımı açısından baktığımızda da her sınıf nesnesi için bir göstericilik bellek alanı kullanılıyor. Ayrıca türetme hiyerarşisi içindeki her sınıf için sanal işlev tablosu olarak kullanılan veri yapısının konumlandırılacağı bellek alanı gerekiyor. Özellikle küçük veriler taşıyan küçük sınıf nesneleri için bu bellek maliyeti bazen istenmiyor.  Performans kritik uygulamalarda sanal işlevlerin faydalarını bir ölçüde bize sağlayacak ancak maliyeti azaltacak bazı başka çözümler var. İşte CRTP bunuı sağlayan örüntülerden (pattern) biri.

CRTP, C++‘ta sık karşımıza çıkan örüntülerden biri. Bu teknik 1980’li yıllarda  “F-bounded quantification” olarak isimlendirilmiş.  1995 yılında Jim Coplien örüntüye bu ismi vermiş. O tarihten bugüne örüntü bu isimle biliniyor.   Türkçeye kelime kelime çevirirsek “meraklıca yinelenen şablon örüntüsü” diyebiliriz. (demesek daha iyi olur gibi geliyor.)
Şimdi örüntümüze bakalım. Taban sınıf olarak kullanılacak bir sınıf şablonumuz var ve türemiş sınıfların her biri bu şablon sınıfın türemiş sınıf açılımından türetiliyor:

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
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
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
İlginç bir yapı değil mi? A sınıfını bir sınıf şablonu olan B‘nin, B<A> açılımından türetiyoruz. Dilin kurallarını çiğneyen hiç bir durum yok. Yukarıdaki örnekte şu nokta özellikle dikkatimizi çekmeli:

void Base<Der>::implementation()
1
void Base<Der>::implementation()
işlevi  Base sınıfından türetilen Der sınıfının bildiriminden önce (yani derleyicinin bu sınıfı bilmesinden önce) bildirilmesine karşın işlevin kodu Der sınıfının bilinmesinden sonra yazılacak.

Dikkatimizi çekmesi gereken ikinci bir nokta da taban sınıfın interface üye işlevi içinde yapılan

static_cast<Derived *>(this)
1
static_cast<Derived *>(this)
dönüşümü. Dilin kurallarına göre bu dönüşümün geçerli  olabilmesi için Base ve Derived sınıflarının aynı hiyerarşi içinde bulunması gerekiyor. Bu dönüşümle Base sınıfı türünden nesnenin adresini (Derived *) türüne dönüştürüyor ve tür dönüştürme işlecinden elde edilen adresle türemiş sınıfının implementation isimli işlevi çağırıyoruz. Derleyici bu durumda implementation ismini türemiş sınıfın isim alanında arıyor.

Bu teknik dinamik çok biçimliliğin maliyetinden kaçınarak ve çoğu zaman ilave esneklik kazandırarak sanal işlevlerin etkilerine benzer bir yapı oluşturuyor. CRTP‘nin bu özel kullanımı bazı kişiler tarafından “statik çokbiçimlilik” ya da “simüle edilmiş dinamik bağlama” olarak isimlendiriliyor. Windows’un ATL ve WTL kütüphanelerinde bu teknik yoğun olarak kullanılıyor. Tekniği anlamak için, sanal işlevleri olmayan bir sınıfı kafanızda canlandırın. Taban sınıf başka işlevleri kendi öğe işlevleri yoluyla çağırıyor. Taban sınıftan bir türetme yaptığımızda taban sınıfın tüm veri öğelerini ve öğe işlevlerini kalıtım yoluyla almış oluyoruz.

Şimdi bir örnek üzerinde dinamik çokbiçimliliği statik çok biçimliliğe dönüştürelim. Aşağıdaki gibi bir dinamik çokbiçimliliğe sahip sınıf hiyerarşimiz olsun:

#include <iostream>
#include <string>

using namespace std;

class Pet
{
public:
	void makeSound()
	{
		std::cout << getSound() << endl;
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
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
#include <iostream>
#include <string>
 
using namespace std;
 
class Pet
{
public:
	void makeSound()
	{
		std::cout << getSound() << endl;
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
Şimdi bu yapıyı CRTP örüntüsüne dönüştürelim:

#include <iostream>
#include <string>

using namespace std;

template <typename T>
class Pet
{
	const T& thisObject() { return *static_cast<const T*>(this); }

public:
	void makeSound()
	{
		std::cout << thisObject().getSound() << "\n";
	}

	// Sanal bir işlev bildirilmiyor. Ancak bu sınıf şablonu T türüne ilişkin 
	// std::string T::getSound() const" imzalı bir işlev bulunacağına göre yazılıyor.
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
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
#include <iostream>
#include <string>
 
using namespace std;
 
template <typename T>
class Pet
{
	const T& thisObject() { return *static_cast<const T*>(this); }
 
public:
	void makeSound()
	{
		std::cout << thisObject().getSound() << "\n";
	}
 
	// Sanal bir işlev bildirilmiyor. Ancak bu sınıf şablonu T türüne ilişkin 
	// std::string T::getSound() const" imzalı bir işlev bulunacağına göre yazılıyor.
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
Pet sınıfının private bölümünde tanımlanan

const T& thisObject() { return *static_cast<const T*>(this); }
1
const T& thisObject() { return *static_cast<const T*>(this); }
işlevi tür dönüştürme kodlarını daha kolay yazabilmemiz için oluşturduğumuz bir yardımcı yalnızca. C++’ın işlev şablonlarına ilişkin önemli bir kuralı da hatırlamanın tam zamanı. Bir işlev çağrısı yapılmadığı sürece derleyici bir işlev şablonundan ya da sınıf şablonunun üye işlevinden bir kod üretmeyecek.  getSound işlevlerinin kodunun üretilmesini tetikleyen main işlevi içinde yapılan çağrılar:

mycat.makeSound();
mydog.makeSound();
1
2
mycat.makeSound();
mydog.makeSound();
Şüphesiz iki yapının ayrı ayrı avantajları ve dezavantajları var. CRTP kullandığımız yapıda:
Ne taban sınıf nesneleri içine gömülü bir gösterici ne de çalışma zamanında bellekte sınıflar için oluşturulmuş bir işlev tablosu tutuluyor.
İşlev çağrısı doğrudan yapılıyor. Yani iki ayrı içerik alma işlemi yok.
Ayrıca dinamik çokbiçimlilikteki kurallardan farklı olarak taban sınıfın işlevinin imzasıyla türemiş sınıfın işlevinin imzasının aynı olması gerekmiyor.
Diğer taraftan taban sınıftan yapılan her bir türetme için hem taban sınıf için hem de türemiş sınıflar için farklı kodlar oluşturulduğundan statik çok biçimlilik sağlayan bu yapıda oluşturulan kod miktarı daha fazla olma eğiliminde.
Hangi işlevin çağrıldığı programın çalışma zamanında değil de derleme zamanında belli olduğu için bu yapının dinamik çok biçimliliğe göre daha kısıtlı bir kullanımı var.
Dinamik çokbiçimlilikteki gibi katı bir tür kontrolü yok.

CRTP‘nin kullanımı bu örneklerle sınırlı değil. Bu serinin diğer yazılarında CRTP‘nin farklı amaçlarla kullanımını da inceleyeceğiz.
