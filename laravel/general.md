# General Laravel Tips

### Quickly inspect return values

```php
public function myFunction()
{
    return tap(User::admins()->get(), 'logger');
}
```

### Document all tables and fields in a markdown file
```php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Route;
use Illuminate\Support\HtmlString;
use Illuminate\Support\Str;
use League\CommonMark\Extension\HeadingPermalink\HeadingPermalinkExtension;
use League\CommonMark\GithubFlavoredMarkdownConverter;

Route::get('/', function () {
    $markdown = (new GithubFlavoredMarkdownConverter([
        'heading_permalink' => [
            'html_class' => '',
            'id_prefix' => '',
            'fragment_prefix' => '',
            'title' => '',
            'symbol' => '',
        ],
    ]))->getEnvironment()->addExtension(new HeadingPermalinkExtension);

    $md = <<<EOT
        # NABE Datenbank
        Eine Dokumentation aller Tabellen in NABE\n\n
        EOT;
    $tableNames = collect(DB::select('SHOW TABLES'))
        ->map(fn ($item) => $item->Tables_in_nabetables)
        ->split(5);
    $maxColumnHeight = $tableNames->first()->count();
    $tableNames = $tableNames->map(fn ($item) => $item->pad($maxColumnHeight, ''))->toArray();

    $md .= <<<EOT
        ## Inhalt
        |  |  |  |  |  |
        |--|--|--|--|--|\n
        EOT;
    for ($i=0; $i < $maxColumnHeight; $i++) {
        $line = array_column($tableNames, $i);
        $md .= "| [{$line[0]}](#" . Str::of($line[0])->lower() . ") | [{$line[1]}](#" . Str::of($line[1])->lower() . ") | [{$line[2]}](#" . Str::of($line[2])->lower() . ") | [{$line[3]}](#" . Str::of($line[3])->lower() . ") | [{$line[4]}](#" . Str::of($line[4])->lower() . ") |\n";
    }

    $md .= "\n## Tabellen\n";
    collect(DB::select('SHOW TABLES'))
        ->map(fn ($item) => $item->Tables_in_nabetables)
        ->each(function ($tableName) use (&$md) {
            $md .= <<<EOT
                ### {$tableName}
                *Hier Beschreibung der Tabelle einfügen*\n
                | Feld | Typ | Beschreibung |
                |------|-----|--------------|\n
                EOT;
            collect(DB::select("SHOW COLUMNS FROM {$tableName}"))
                ->map(fn ($table) => [$table->Field, $table->Type])
                ->eachSpread(function ($field, $type) use (&$md) {
                    $md .= "| {$field} | {$type} | Beschreibung |\n";
                });
            $md .= "\n";
        });
    return new HtmlString((string) $markdown->convertToHtml($md));
});
```
