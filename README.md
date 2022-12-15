# JavaScript - Proxy Design Pattern

Proxy je dizajn patern koji pruža zamenu za drugi objekat i kontroliše pristup tog drugog objekta.

## Upotreba Proxija

U objektno orijentisanom programiranju, objekti obavljaju posao koji je predstavljen kroz interfejs (svojstva i metode). Klijenti koji zatraže request od ovih objekata očekuju da se ovaj posao obavi brzo i učinkovito. Međutim, postoje situacije u kojima je objekat ozbiljno ograničen i ne može završiti svoj posao. Obično se to događa kada postoji zavisnost (dependency) o udaljenom (remote) resursu (npr. zbog kašnjenja na mreže) ili kada objektu treba dugo vremena da se učita.

U ovakvim situacijama primenjujemo proxy objekat koji "zamenjuje" osnovni objekat. Proxy prosleđuje zahtev ciljanom objektu. Interfejs proxy objekta isti je kao i osnovni objekat i klijenti možda nisu ni svesni da imaju posla sa proxy-jem, a ne pravim objektom.

## Diagram

<img width="500" alt="proxy-pattern" src="https://user-images.githubusercontent.com/21141150/207812719-ceafff39-94ce-4637-9afa-5ed667eca82f.png">

## Učesnici

Objekti koji učestvuju u ovom paternu su:

**Client** -- U primeru: run() funkcija
- poziva Proxy i zahteva određenu operaciju


**Proxy** -- U primeru: GeoProxy
- pruža interfejs sličan stvarnom objektu
- održava referencu koja proxyju omogućava pristup stvarnom objektu
- obrađuje zahteve i prosleđuje ih stvarnom objektu

**RealSubject** -- U primeru: GeoCoder
- definiše stvarni objekat od koga se traži usluga.


## Primer

Objekat **GeoCoder** simulira uslugu Google Maps. Za GeoCoder šaljete lokaciju (mesto na planeti) i on će vratiti njegovu širinu/dužinu (lat/lng). Naš GeoCoder može razlikovati samo 4 lokacije, ali u stvarnosti ih ima mnogo više, jer uključuje države, pokrajine, gradove i ulice.

Programer je odlučio implementirati Proxy objekat jer je **GeoCoder** relativno spor. Proxy objekat se zove **GeoProxy**. Poznato je da dolazi mnogo istih zahteva (za istu lokaciju). Kako bi ubrzao stvari, GeoProxy kešira često tražene lokacije. Ako lokacija nije već učitana u keš, zahtev odlazi na pravi GeoCoder servis i učitava rezultate u keš memoriju.

Šalju se zahtevi za lokaciju nekoliko gradova, ali mnogi od njih su vezani za isti grad. GeoProxy poziva GeoCoder, a potom ga učitava u keš memoriju. Na kraju je GeoProxy obradio 11 zahteva, ali je pozvao GeoCoder samo 4 puta. Primetite da klijent nema predstavu o proxy objektu (poziva isti interfejs sa standardnom metodom getLatLng).

```
function GeoCoder() {

    this.getLatLng = function (address) {

        if (address === "Sombor") {
            return "45.7733° N, 19.1151° E";
        } else if (address === "NoviSad") {
            return "45.2396° N, 19.8227° W";
        } else if (address === "Nis") {
            return "43.3209° N, 21.8954° E";
        } else if (address === "BanjaLuka") {
            return "44.7722° N, 17.1910° E";
        } else {
            return "";
        }
    };
}

function GeoProxy() {
    var geocoder = new GeoCoder();
    var geocache = {};

    return {
        getLatLng: function (address) {
            if (!geocache[address]) {
                geocache[address] = geocoder.getLatLng(address);
            }
            console.log(address + ": " + geocache[address]);
            return geocache[address];
        },
        getCount: function () {
            var count = 0;
            for (var code in geocache) { count++; }
            return count;
        }
    };
};

function run() {

    var geo = new GeoProxy();

    // GeoLocation zahtevi

    geo.getLatLng("NoviSad");
    geo.getLatLng("Sombor");
    geo.getLatLng("Sombor");
    geo.getLatLng("Sombor");
    geo.getLatLng("Sombor");
    geo.getLatLng("Nis");
    geo.getLatLng("Nis");
    geo.getLatLng("Nis");
    geo.getLatLng("Nis");
    geo.getLatLng("BanjaLuka");
    geo.getLatLng("BanjaLuka");

    console.log("\nCache size: " + geo.getCount());
}

// → Novi Sad: 45.2396° N, 19.8227° W
// → Sombor: 45.7733° N, 19.1151° E
// → Sombor: 45.7733° N, 19.1151° E
// → Sombor: 45.7733° N, 19.1151° E
// → Sombor: 45.7733° N, 19.1151° E
// → Nis: 43.3209° N, 21.8954° E
// → Nis: 43.3209° N, 21.8954° E
// → Nis: 43.3209° N, 21.8954° E
// → Nis: 43.3209° N, 21.8954° E
// → Banja Luka: 44.7722° N, 17.1910° E
// → Banja Luka: 44.7722° N, 17.1910° E
// → Cache size: 4
```
