# XML Feed Generátor (Laravel package)
Eloquent model / DB table alapú XML feed generátor

### Példán keresztül az összes opció bemutatása
Input adatok:
```
$savePath = 'app/public/feed/valami.xml';  //ha nem '/' -rel kezdődik, akkor storage_path(...) -ba teszi
$eloquentClassOrTable = Job\Models\Job::class;
$where = [['is_direct', '=', 1], ['is_active', '=', 1], ['hidden', '=', 0]];
$header = '<?xml version="1.0" encoding="UTF-8" ?><jobs>';
$footer = '</jobs>';
$isStrict = false; //ha egy sor egyik mezője nem teljesít egy kritériumot (required, min: etc) => TRUE: megszakad, és hibát dob FALSE: átugorja az aktuális sort

$anItem = [
    "job" => [
        "_attributes" => [
            "id" => "id",
            "country" => "getCountry()",
        ],
        "valamicucc" => [
            "_attributes" => [
                "id" => "id",
                "country" => "name",
            ],
            "_data|cdata|bool:no:yes" => " 'valamistatikus' ",
        ],
        "referencenumber|required|cdata|number_format:2:,:." => "id",
        "title|required|cdata|max:20" => "name",
        "company|required|cdata" => "company_name",
        "city|cdata|required" => "workplaceCity()",
        "dateposted|cdata|required|datetime:Y-m-d" => "created_at",
        "url|cdata|required" => "getUrl() + '?utm_source=talent' ",
    ],
];

```

Generálás:
```
  app(FeedGenerator\Contracts\GenerateFeedServiceInterface::class)->generateFeed(
      $savePath,
      $eloquentClassOrTable,
      $where,
      $header,
      $anItem,
      $footer,
      $isStrict 
  );
```

Output:
``` 
<?xml version="1.0" encoding="UTF-8" ?>
<jobs>
  <job id="9668786" country="Magyarország" >
    <valamicucc id="9668786" country="Lakatos, hegesztő" >
      <![CDATA[no]]>
    </valamicucc>
    <referencenumber>
      <![CDATA[9668786]]>
    </referencenumber>
    <title>
      <![CDATA[Lakatos, hegesztő]]>
    </title>
    <company>
      <![CDATA[Andritz Kft]]>
    </company>
    <city>
      <![CDATA[Tiszakécske]]>
    </city>
    <dateposted>
      <![CDATA[2022-10-05]]>
    </dateposted>
    <url>
      <![CDATA[https://jobinfo.loc/allas/lakatos-hegeszto/tiszakecske/9668786?utm_source=talent]]>
    </url>
  </job>
  (...)
</jobs>
```

### $anItem opciók kifejtve
```
"item_tag" => [
  field_tag|opciók => db_mezo + methodNeve() + 'custom string'
]
```

#### field_tag|opciók
**FONTOS: Ha nem Eloquent model, hanem táblanév van megadva, akkor értelemszerűen method nevet nem tud feldolgozni**  
- cdata `<![CDATA[...]]>` -ba teszi
- required  (ha isStrict akkor error, ha !isStrict, akkor átugorja az aktuális elemet)
- min:30 (ha isStrict akkor error, ha !isStrict, akkor átugorja az aktuális elemet)
- max:100 (levágja a végét)
- strip_tags
- bool:ÉRTÉK_HA_FALSE:ÉRTÉK_HA_TRUE
- datetime:FORMAT (lehet sima date típusú mező is, nem csak datetime)
- number_format:paraméter2:paraméter3:paraméter4 (php number_format fg.)

#### db_mezo + methodNeve() + 'custom string'
- `+` -val elválasztva: `"getUrl() + '?utm_source=talent'"`
- ha van benne `()` akkor `$eloquentModel->methodNeve()` hívja meg, ha ez `object`-tel tér vissza, tehát pl `belongsTo()` akkor automatikusan `->first()->name`
- ha szimpla idézőjelben van `'?utm_source=talent'` akkor azt simán hozzáfűzi stringként
- ha a fentiek közül egyik sem, akkor az DB mezőt hívja meg `"created_at"`



