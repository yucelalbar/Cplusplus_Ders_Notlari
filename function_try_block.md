#### işlev try bloğu (function try block)

Bir sınıfın kurucu işlevi görevini yerine getiremeyeceğini anladığında ne yapmalı? Bunu herkes biliyor, bir hata nesnesi göndermeli. 
Peki ya eğer kurucu işlevin  öğe ilk değer verme listesiyle _(member initializer list)_ hayata getirilen sınıf öğelerinin kurucu işlevlerinin birinden hata nesnesi 
gönderilirse ne olacak? 
Bu durumda programın akışı sınıfımızın kurucu işlevinden çıkacak değil mi? Aşağıdaki koda bir bakalım:
```
class Member {
	//
public:
	Member(int);
	//
};


class Owner {
	Member m;
public:
	Owner() : m(0) 
	{
		///
	}
};
```
Owner sınıfının Member sınıfı türünden bir veri öğesi var ve Owner sınıfının varsayılan kurucu işlevi bu öğeyi, öğe ilk değer verme listesiyle hayata getiriyor. Owner sınıfının kurucu işlevi çağrıldığında Member sınıfının int parametreli kurucu işlevinden bir hata nesnesi gönderilirse ne olur? Öncelikle C++’ın çok net bir kuralını bilmemiz gerekiyor. Eğer bir sınıf nesnesinin kurucu işlevinin kodu tamamen yürütülmez ise derleyici bu sınıf nesnesi için sonlandırıcı işlevi çağırmaz. Bu kuralın gerekliliği çok açık: Sınıfın kurucu işlevi içinden, hata nesnesinin gönderildiği kaynak kod noktasından sonra birtakım kaynakların elde edildiğini düşünelim. Bu durumda sonlandırıcı işlev çağrılsaydı, hata nesnesinin gönderilmesi durumunda  sonlandırıcı işlev edinilmemiş kaynakları geri verme girişiminde bulunacak bu da  çalışma zamanı hatalarına neden olacaktı. Yani en azından şöyle bir güvencemiz var: Member sınıfının kurucu işlevinden bir hata nesnesinin gönderilmesi durumunda, ne Member ne de Owner sınıfının kurucu işlevinin kodu tamameen yürütülmüş olacak ve her iki sınıfın da sonlandırıcı işlevi çağrılmayacak.

Sorumuza geri dönelim: Sınıfımızın ilk değer verme listesinin yürütülmesi sürecinde bir hata nesnesi gönderilirse ne yapmalıyız? Eğer yanıtınız hiç bir şey yapmamalıyız ise bu yanıt çoğunlukla yanlış. En azından yapmamız gereken gönderilen hata nesnesini yakalayarak kurucu işlevimizin hata durumunu loglamak ve yakalanan hata nesnesini yeniden göndermek (rethrow). Bazı durumlarda rethrow işlemiyle yakalanan hata nesnesini yeniden göndermek yerine, yukarıdaki daha yüksek seviyeli kodlara durumu daha iyi anlatacak, kendi belirlediğimiz bir türden hata nesnesi göndermeyi de tercih edebiliriz.

Peki hata nesnesini yakalamak için ne yapacağız? Kurucu işlevimizin kodunun tamamını bir try bloğu içine alsak bile öğe ilk değer verme listesinden gönderilecek hata nesnelerini yakalayamayız. Çünkü öğe ilk değer verme listesi kodları try bloğu içinde yer almıyor:

class Member {
	//
public:
	Member(int);
	//
};


class Owner {
	Member m;
public:
	Owner(int x) : m(x) 
	{
		try{
			//kurucu işlevin kodu
		}
		catch (...) {
			///
		}
	}
};
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
class Member {
	//
public:
	Member(int);
	//
};
 
 
class Owner {
	Member m;
public:
	Owner(int x) : m(x) 
	{
		try{
			//kurucu işlevin kodu
		}
		catch (...) {
			///
		}
	}
};
Yukarıdaki kodda Owner sınıfının kurucu işlevinin tüm kodlarının try bloğu içinde yer aldığını düşünelim. Member sınıfının kurucu işlevinden bir hata nesnesi gönderilirse programın akışı catch bloğuna girmeyecek.

C++ dili öğe ilk değer verme listesinden gönderilebilecek hata nesnelerinin yakalanmasına olanak verecek özel bir try bloğu oluşturmaya olanak sağlıyor.  Böyle try bloklarına işlev try bloğu deniyor. İşlev try bloğu hem kurucu işlevin öğe ilk değer verme listesini hem de ana bloğunu kapsayan bir try bloğu olarak düşünülebilir:

class Member {
	//
public:
	Member(int);
	//
};


class Owner {
	Member m;
public:
	Owner(int x) try : m(x)  
	{
		//kurucu işlevin kodu
	}
	catch (...) {
		///
	}
};
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
class Member {
	//
public:
	Member(int);
	//
};
 
 
class Owner {
	Member m;
public:
	Owner(int x) try : m(x)  
	{
		//kurucu işlevin kodu
	}
	catch (...) {
		///
	}
};
Şimdi de işlev try bloklarıyla ilgili kodlama açısından önemli olabilecek noktalara değinelim:

Bir sınıfın öğe ilk değer verme listesinden bir hata nesnesi gönderilirse, ilk değer listesiyle hayata getirilmiş (yani kurucu işlev kodunun tamamı çalıştırılmış) öğe nesnelerin sonlandırıcı işlevleri, programın akışının işlev try bloğuna ilişkin catch bloğuna girmesinden önce çağrılır.

Kurucu işlevin parametre değişkenleri sınıf türlerinden ise bu parametre değişkenleri için çağrılacak olan kopyalayan kurucu işlev, taşıyan kurucu işlev ya da sonlandırıcı işlevlerden gönderilen hata nesneleri işlev try bloğu ile yakalanamazlar.

Programın akışı işlev try bloğunu izleyen bir catch bloğuna girdiğinde bu catch bloğu içinde program sonlandırılmaz ya da catch bloğundan bir hata nesnesi (yeniden) gönderilmez ise derleyicinin ürettiği kod ile otomatik olarak hata nesnesi yeniden gönderilir (rethrow).

İşlev try bloklarının genel kullanım amacı kurucu işlevlere yönelik olsa da herhangi bir global işlev ya da bir sınıfın üye işlevi de işlev try bloğuna sahip olabilir. Örneğin dilersek main işlevi için bir işlev try bloğu oluşturabiliriz:

int main() 
try {
	//
}
catch (...) {
	///
}
1
2
3
4
5
6
7
int main() 
try {
	//
}
catch (...) {
	///
}
Normal try bloklarında olduğu gibi işlev try bloğu içinde tanıtılan isimler de try bloğunu izleyen catch blokları içinde bilinmezler. Ancak işlevin parametre değişkenleri catch bloklarında da bilinirler ve  programın  akışının bir catch bloğuna girmesi durumunda parametre değişkenleri catch bloğunun kodunun çalışması bitene kadar hayatta kalırlar:

#include <iostream>

void func(int x)
try {
	throw 1;
}
catch (...) {
	std::cout << "x = " << x << "\n";
	++x;
	std::cout << "x = " << x << "\n";
}

int main()
{
	func(10);

	return 0;
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
#include <iostream>
 
void func(int x)
try {
	throw 1;
}
catch (...) {
	std::cout << "x = " << x << "\n";
	++x;
	std::cout << "x = " << x << "\n";
}
 
int main()
{
	func(10);
 
	return 0;
}
Bir işlev try bloğunu izleyen catch bloğu içinde bir return deyimi yer alabilir. Böyle bir return deyimi try bloğuna konu işlevin return deyimidir. catch bloğu içinde bir return deyimi yok ise programın akışı catch bloğunun sonuna geldiğinde işlevin çalışan kodu sonlanmış olur. Eğer işlev void değilse yani işlevin bir geri dönüş değeri türü var ise programın akışının catch bloğunun sonuna kadar gelmesi tanımsız davranıştır.

İsim alanı bilinirlik alanında (namespace scope) oluşturulmuş statik ömürlü bir nesnenin kurucu işlevinden gönderilecek bir hata nesnesi main işlevi için oluşturulacak olan bir işlev try bloğu içinde yakalanamaz. statik ömürlü global nesnelerin kurucu işlevleri main çağrılmadan önce çağrılmaktad
