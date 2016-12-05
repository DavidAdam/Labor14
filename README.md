# Labor14

## Bevezető

Android 6.0 (API level 23) verziótól kezdve a felhasználó futásidőben adhatja meg az alkalmazás engedélyeit, és nem az alkalmazás telepítésekor vagy frissítésekor. Dönthet úgy, hogy bizonyos engedélyeket nem ad meg egy alkalmazásnak, így nagyobb fokú irányítás kerül a kezébe. Az alkalmazásengedélyeket később bármikor módosíthatja az alkalmazásbeállítások képernyőn.

Az engedélyek két kategóriába vannak sorolva: normal/dangerous.
A normal kategóriába tartozó engedélyek nem jelentenek közvetlen kockázatot a felhasználó érzékeny adataira, ezeket az engedélyeket a rendszer automatikusan megadja.

A dangerous kategóriába tartozó engedélyek lehetőséget adhatnak az alkalmazásnak, hogy a felhasználó érzékeny adataihoz hozzáférjen. Ebben az esetben a felhasználónak kell megadni az engedélyt az alkalmazás számára.

https://developer.android.com/guide/topics/security/permissions.html#normal-dangerous

Minden esetben meg kell adni a normal és a dangerous engedélyeket a manifest fájlban, de ennek hatása eltér a rendszer verziójától és a target sdk szinttől függően:

Ha az eszköz Android 5.1 vagy alacsonyabb verziót futtat, VAGY az alkalmazás target SDK szintje 22 vagy kisebb, akkor a rendszer telepítéskor kéri el az összes engedélyt, és ha a felhasználó nem fogadja el, akkor a telepítés leáll.

Ha az eszköz Android 6.0 verzióval rendelkezik ÉS az alkalmazás target SDK szintje 23 vagy nagyobb, akkor az alkalmazás a futása során fogja elkérni a dangerous kategóriába tartozó engedélyeket, a normal engedélyeket pedig a rendszer automatikusan megadja. A felhasználó bármely engedélyt megadhat, vagy letilthat, emiatt az alkalmazás limitált funkcionalitással futhat tovább, melyet megfelelően kell kezelni.

## Jogosultság ellenőrzése
Amennyiben az alkalmazás funkciójának egy veszélyes engedélyre van szüksége, akkor minden esetben ellenőrízni kell még a funkció indítása előtt, hogy rendelkezik-e az engedéllyel, hiszen az engedélyeket a felhasználó bármikor módosíthatja.
ContextCompat.checkSelfPermission()

## Jogosultság elkérése
Az Android rendszer számos metódust biztosít egy jogosultság elkérésére. Ezeket meghívva egy nem testreszabható dialógust dob fel a rendszer.
ActivityCompat.requestPermissions()

## Jogosultság magyarázata
Egyes esetekben szükséges lehet a felhasználót tájékoztatni, hogy miért kér az alkalmazás veszélyes engedélyeket.
ActivityCompat.shouldShowRequestPermissionRationale()

## Feladat

## Kezdő lépések

A labor során egy egyszerű telefonkönyv alkalmazást kell elkészíteni. Az alkalmazás listázni tudja a telefonon tárolt névjegyeket, majd egy adott elemre kattintva hívást lehet kezdeményezni.

Hozzunk létre egy új Android Studio Projektet PermissionsLabor néven. A Company Domain mező tartalmát töröljük ki és hagyjuk is üresen.

A packagename legyen hu.bme.aut.amorg.examples.permissionslabor A támogatott céleszközök a Telefon és Tablet, valamint a minimum SDK szint az API15: Android 4.0.3

A kezdő projekthez adjuk hozzá egy Empty Activity-t, melynek neve legyen ContactsActivity.

Vegyük fel a RecyclerView komponens függőségét, illetve állítsuk a targetSDK-t 23 vagy nagyobbra a build.graddle(module:app) fájlban, majd nyomjuk meg a Sync Now gombot. Amennyiben nincs 23, vagy magasabb SDK telepítve a gépre, akkor frissítsük az SDK Manager segítségével a szükséges komponenseket.


```java
dependencies {
...
compile 'com.android.support:recyclerview-v7:25.0.0'
...
}
```

## Felhasználói felület

Első lépésként készítsük el az alkalmazás felhasználói felületét XML erőforrásból. A felület egyetlen RecyclerView komponensből fog állni, mely az eszközön tárolt névjegyeket fogja megjeleníteni.

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_contacts"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="hu.bme.aut.amorg.examples.permissionslabor.ContactsActivity">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/contactsRV"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</RelativeLayout>
```

A ContactsActivity-ben szerezzünk referenciát a RecyclerView-ra, és emeljük ki mezőbe.

```java
private RecyclerView contactsRV;
```

onCreate() metódusba:
```java
contactsRV = (RecyclerView) findViewById(R.id.contactsRV);
```

Készítsük el a Contact osztályt, mely az eszközön található névjegyeket fogja reprezentálni. Az egyszerűség kedvéért most csak a név és telefonszám adatokat tároljuk el benne.

```java
public class Contact {
    private String contactName;
    private String contactNumber;

    public String getContactName() {
        return contactName;
    }

    public void setContactName(String contactName) {
        this.contactName = contactName;
    }

    public String getContactNumber() {
        return contactNumber;
    }

    public void setContactNumber(String contactNumber) {
        this.contactNumber = contactNumber;
    }
}
```

Készítsük el a listát feltöltő adaptert ContactsAdapter néven, adapter csomagba.

```java
public class ContactsAdapter extends RecyclerView.Adapter<ContactsAdapter.ContactViewHolder> {
    
    private List<Contact> contactList;
    private Context mContext;

    public ContactsAdapter(List<Contact> contactList, Context mContext) {
        this.contactList = contactList;
        this.mContext = mContext;
    }

    @Override
    public ContactViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(mContext).inflate(R.layout.contact_item, null);
        ContactViewHolder contactViewHolder = new ContactViewHolder(view);
        return contactViewHolder;
    }

    @Override
    public void onBindViewHolder(final ContactViewHolder holder, int position) {
        Contact contact = contactList.get(position);
        holder.tvContactName.setText(contact.getContactName());
        holder.tvPhoneNumber.setText(contact.getContactNumber());
    }


    @Override
    public int getItemCount() {
        return contactList.size();
    }

    public static class ContactViewHolder extends RecyclerView.ViewHolder {
        View container;
        ImageView ivContactImage;
        TextView tvContactName;
        TextView tvPhoneNumber;

        public ContactViewHolder(View itemView) {
            super(itemView);
            container = itemView.findViewById(R.id.container);
            ivContactImage = (ImageView) itemView.findViewById(R.id.ivContactImage);
            tvContactName = (TextView) itemView.findViewById(R.id.tvContactName);
            tvPhoneNumber = (TextView) itemView.findViewById(R.id.tvPhoneNumber);
        }
    }
}
```

Az adapter az onCreateViewHolder() metódusában hivatkozik a listaelem felületleírójára, hozzuk létre a hiányzó contact_item xml erőforrást:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:background="#dddddd">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <ImageView
            android:id="@+id/ivContactImage"
            android:layout_width="55dp"
            android:layout_height="55dp"
            android:layout_marginLeft="10dp"
            android:layout_marginStart="10dp"
            android:src="@drawable/ic_contact_phone_black_48dp"/>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical"
            android:gravity="center_vertical">

            <TextView
                android:id="@+id/tvContactName"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginLeft="10dp"
                android:layout_marginStart="10dp"
                android:textSize="16sp"
                android:textColor="@android:color/primary_text_light"
                android:text="@string/name"/>

            <TextView
                android:id="@+id/tvPhoneNumber"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginLeft="10dp"
                android:layout_marginStart="10dp"
                android:textSize="14sp"
                android:textColor="@android:color/primary_text_light"
                android:text="@string/phone"/>
        </LinearLayout>
    </LinearLayout>
</RelativeLayout>
```

Adjuk hozzá a két hiányzó szöveges erőforrást, és másoljuk be a drawables mappába a hiányzó képet!

strings.xml-be:
```xml
...
<string name="name">Name</string>
<string name="phone">Phone</string>
...
```

drawables mappába:
LINK A KÉPHEZ!

A névjegyek megjelenítéséhez az utolsó lépés az adapter pélányosítása, és beállítása a recyclerview komponenshez. Szükség van az eszközön tárolt névjegyek megszerzésére, ehhez adjuk hozzá a ContactsActivity-be az alábbi metódust:

```java
private List<Contact> getAllContacts() {
    List<Contact> contactList = new ArrayList();
    ContentResolver contentResolver = getContentResolver();
    Cursor cursor = contentResolver.query(ContactsContract.Contacts.CONTENT_URI, null, null, null, ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME + " ASC");
    if (cursor.getCount() > 0) {
        while (cursor.moveToNext()) {
            int hasPhoneNumber = Integer.parseInt(cursor.getString(cursor.getColumnIndex(ContactsContract.Contacts.HAS_PHONE_NUMBER)));
            if (hasPhoneNumber > 0) {
                String id = cursor.getString(cursor.getColumnIndex(ContactsContract.Contacts._ID));
                String name = cursor.getString(cursor.getColumnIndex(ContactsContract.Contacts.DISPLAY_NAME));

                Contact contact = new Contact();
                contact.setContactName(name);

                Cursor phoneCursor = contentResolver.query(
                        ContactsContract.CommonDataKinds.Phone.CONTENT_URI,
                        null,
                        ContactsContract.CommonDataKinds.Phone.CONTACT_ID + " = ?",
                        new String[]{id},
                        null);
                if (phoneCursor.moveToNext()) {
                    String phoneNumber = phoneCursor.getString(phoneCursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
                    contact.setContactNumber(phoneNumber);
                }
                phoneCursor.close();
                contactList.add(contact);
            }
        }
    }
    return contactList;
}
```

Ez után a kapott névjegylistával példányosítsuk az adaptert, és állítsuk be a recyclerview komponenshez.
ContactsActivity onCreate() metódusába:

```java
ContactsAdapter contactsAdapter = new ContactsAdapter(getAllContacts(), this);
contactsRV.setLayoutManager(new LinearLayoutManager(this));
contactsRV.setAdapter(contactsAdapter);
```

Névjegyek olvasásához szükséges engedély a manifest-be:
```xml
<uses-permission android:name="android.permission.READ_CONTACTS" />
```

Egyelőre semmilyen jogosulságkezelést nem valósítottunk meg a kódban, ezért az alkalmazás pillanatnyi állapotának kipróbálásához Android 6.0 előtti verzióra van szükség, különben hibát kapunk az indulás során.

Próbáljuk ki az alkalmazást 6.0/API 23 előtti verzióval rendelkező eszközön!
Amennyiben az eszközön nincsenek névjegyek, adjunk hozzá legalább egyet telefonszámmal ellátva.

KÉP KÉP KÉP KÉP
