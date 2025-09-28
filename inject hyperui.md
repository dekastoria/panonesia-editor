(A) Ringkasan (bahasa sederhana)

Data & Relasi

Kamu punya daftar Panorama (Scene): pano1, pano2, …

Tiap panorama punya banyak Popup (teks, gambar, galeri, PDF, video, audio, YouTube, HTML).

Kunci popup otomatis: pop-<scene_slug>-<type>-<nnn>, mis. pop-pano1-text-001.

Editor di Filament

Warna/komponen ikut tema Hasnayeen—kita atur posisi/layout saja.

Pada halaman Edit Popup: form kiri (isian), panel kanan: HyperUI-like preview dengan toggle Preview | HTML, tombol Mobile/Tablet/Desktop/Full/Rotate.

Ada tombol aksi: Build Draft (lihat hasil produksi), Build (tulis inject/dist/registry), Export ZIP (satu paket untuk tur).

Build & Output

Build membuat:

public/inject/<key>.html

public/dist/core.css (sekali)

public/dist/popup-<key>.css (opsional per popup)

public/inject/registry.json (kelompok per scene)

public/loader.js

Export ZIP menyertakan inject + dist + assets media (gambar/pdf/video/audio) yang dipakai popup.

Preview

Mode A (Live): cepat (Tailwind CDN), auto-refresh saat edit.

Mode B (Production): hasil sebenarnya (link ke CSS dist), lewat Build Draft → tampil di tab Production (iframe route /admin/preview/{key}).

3DVista

Panggil openPopup('<key>') untuk munculkan iframe.

Ada helper: listScene('<scene>'), openScenePopup('<scene>','<type>',index).

Acceptance

CRUD jalan, preview jalan, Build menulis file yg benar, Build All sukses, Export ZIP lengkap media, loader berfungsi, performa oke untuk 50–90 popup.

(B) Rincian Runut (end-to-end)
0) Struktur Proyek & Konvensi
public/
  dist/
    core.css
    popup-<key>.css
  inject/
    registry.json
    pop-<scene>-<type>-<nnn>.html
  loader.js
storage/app/preview/
resources/
  views/popup/     (template produksi per tipe)
  views/filament/preview/   (preview Mode A, ala HyperUI, 1 file per tipe jika perlu)
  tour/core.css
  tour/popup.css
tailwind.core.cjs
tailwind.popup.cjs


Jangan pasang warna manual di blade Filament; pakai komponen/kelas Filament agar sinkron tema Hasnayeen.

1) Database & Model
1.1 Migrations

scenes

id, slug (unique, ex: pano1), title (nullable), order (int, default 0), timestamps

popups

id, scene_id (nullable untuk global), key (unique), type (enum: text|img|gal|pdf|yt|video|audio|html|custom)

status (enum: draft|ready, default draft)

props_json (json) — payload per tipe

order (int, default 0), timestamps

1.2 Models

Scene hasMany Popup

Popup belongsTo Scene

$casts['props_json' => 'array']

1.3 Generator Key (otomatis saat create)

Format: pop-<scene_slug>-<type>-<nnn>

nnn = 3 digit urut per (scene, type).

Jika scene_id null → scene_slug = 'global'.

Implementasi: service kecil KeyFactory::for($scene, $type) men-scan Popup by scene+type, ambil suffix terbesar, +1, str_pad 3 digit.

2) Filament UX: Navigasi, Resource, Layout
2.1 Navigasi

ScenesResource (daftar panorama)

Header actions: Add Scene

RelationManager: Popups (list per scene), header action “Add Popup”

PopupsResource

Index: list semua popup (filter by scene/type/status)

Edit Page layout:

Left (Form): meta + dynamic fields per tipe (lihat 3.2)

Right (Section): Preview ala HyperUI:

Toggle Preview | HTML

Tombol Mobile (375) / Tablet (768) / Desktop (1280) / Full / Rotate

Tab Production (iframe /admin/preview/{key}) muncul jika sudah Build Draft

Header actions: Build Draft, Build, Export ZIP (global)

Semua container gunakan Section, Grid, Card bawaan Filament agar warna ikut tema.

3) Form & Tipe Popup
3.1 Tipe & Skema props_json
type	props_json contoh
text	{ "title":"", "body": "markdown/plain" }
img	{ "src":"url", "caption":"" }
gal	{ "items":[ { "src":"url","caption":"" }, ... ] }
pdf	{ "src":"url" }
video	`{ "src":"url.mp4
audio	`{ "src":"url.mp3
yt	{ "videoId":"dQw4w9WgXcQ" }
html	{ "html":"<div>..</div>" } (sanitize saat render)
custom	bebas—tetap di-serialize ke JSON
3.2 Komponen Form (menghindari hard-code warna)

Switch Upload vs URL (mis. Toggle → jika Upload aktif, tampil FileUpload; kalau tidak, tampil TextInput URL).

Validasi acceptedFileTypes & maxSize:

image: image()->maxSize(3072) + dir uploads/images

pdf: application/pdf max 10MB dir uploads/docs

video: video/mp4|video/webm max 50MB dir uploads/videos

audio: audio/mpeg|audio/ogg max 20MB dir uploads/audios

php artisan storage:link → gunakan Storage::url() untuk preview dan build.

Catatan: Untuk URL eksternal (CDN/YouTube) cukup TextInput. Untuk upload, simpan URL jadi string ke props_json (resolve ke Storage::url($path) pada saving hook).

4) Preview Ala HyperUI (Mode A)
4.1 Prinsip

Gunakan ViewField → iframe srcdoc dengan Tailwind CDN.

Toggle: Preview (render komponen) / HTML (tampilkan sumber HTML final yang sama).

Device: tombol Mobile/Tablet/Desktop/Full mengatur iframe.style.width, Rotate tukar W/H.

Kanvas: gunakan wrapper “board” abu-abu tanpa warna custom—pakai util kelas netral (atau biarkan kosong). Fokus ke tata letak, bukan palet.

4.2 Komponen Preview per Tipe

text: heading/body → komponen HyperUI/Tailwind Plus (section minimal).

img: <img class="w-full rounded-lg object-cover">

gal: grid responsif 2–4 kolom.

pdf: <iframe src="..." class="w-full h-[480px]">

video: <video controls poster="..."><source src="..."></video>

audio: <audio controls src="...">

yt: <iframe src="https://www.youtube.com/embed/{videoId}" ...>

html: langsung inject (di Mode A boleh karena sandbox iframe srcdoc; tetap escape di Code view).

Satu blade preview bisa switch by type: builder kecil merakit HTML konten dari props_json lalu injeksi ke template srcdoc.

5) Production Preview (Mode B)

Action “Build Draft”:

Render HTML draft ke storage/app/preview/draft-<key>.html

Build CSS core & optional CSS per popup (lihat §6)

Route /admin/preview/{key} (auth) → return file draft (no-cache)

Di panel Preview, sediakan tab Production → iframe ke route di atas (tambah query ?t={timestamp} utk bust cache)

6) Pipelines Build
6.1 CSS

Core: public/dist/core.css (sekali) via tailwind.core.cjs (+ safelist util umum).

Per Popup (opsional): public/dist/popup-<key>.css via tailwind.popup.cjs dengan --content menunjuk HTML sementara (agar purge spesifik).

Jika ingin super simpel: cukup core.css saja; kalau ingin CSS minimal per popup aktifkan pipeline per-popup.

6.2 HTML

Template produksi per tipe di resources/views/popup/<type>.blade.php

Wrapper generik memuat:

<link rel="stylesheet" href="../dist/core.css">
<link rel="stylesheet" href="../dist/popup-{{ $key }}.css"> <!-- opsional -->


Tombol Close → window.parent?.closePopup?.().

6.3 Registry (group by scene)
{
  "version": 1,
  "scenes": {
    "pano1": {
      "pop-pano1-text-001": { "type":"text", "file":"inject/pop-pano1-text-001.html", "preset":"wide" },
      "pop-pano1-gal-002":  { "type":"gal",  "file":"inject/pop-pano1-gal-002.html",  "preset":"wide" }
    },
    "pano2": { ... }
  }
}


Increment version saat Build All agar cache-busting di loader.

6.4 Media Copy & URL Rewrite (untuk Export ZIP)

Saat Export ZIP:

Scan semua props_json popup status=ready. Ambil URL local storage (prefix /storage/…).

Salin file ke build/assets/... (mirroring subfolder).

Rewrite URL di HTML render untuk paket: dari /storage/... → assets/....

Bundle: inject/*, dist/*, assets/* → tour-assets.zip.

Jika URL eksternal, biarkan.

7) Loader 3DVista
7.1 Fitur

openPopup(key) / closePopup()

Fetch inject/registry.json (no-store)

Buat overlay + panel + iframe inject/<key>.html?v=<version>

ESC/backdrop close, kembalikan fokus

Helper:

listScene(sceneSlug) → array keys

openScenePopup(sceneSlug, type, index) → bangun key pop-<scene>-<type>-<nnn> by order

7.2 Catatan

Izinkan iframe untuk /inject/ di Nginx (hapus X-Frame-Options).

Cache kuat untuk /dist (immutable 1y), no-store untuk registry.json.

8) aaPanel / Deploy

Document root → {project}/public

Pastikan writable: public/inject, public/dist

php artisan storage:link

Node path untuk npx tailwindcss (gunakan full path jika service PHP tak punya PATH)

Nginx:

/dist/ → Cache-Control: public, max-age=31536000, immutable

/inject/ → hapus X-Frame-Options, set Cache-Control: no-cache

CORS hanya jika registry diakses lintas origin

Supervisor/Queue (opsional) untuk Build All besar.

9) Keamanan, Kualitas, Performa

Validasi MIME & ukuran (lihat §3.2)

Sanitasi html/custom (server-side whitelist atau DOMPurify di runtime dalam iframe)

Timeout proses build (Symfony Process) + error surfacing di Filament notifications

Concurrency: lock file sederhana saat Build All (hindari bentrok write)

Versioning registry.json & query param ?v=version untuk iframe dist

Testing:

Unit: key generator, registry builder, URL rewrite

Feature: Build One/All menghasilkan file benar

Browser: Loader ESC/backdrop, fokus kembali

10) Acceptance Test (detil)

CRUD Scene & Popup

Tambah pano1, pano2

Tambah popup text di pano1 → key = pop-pano1-text-001

Tambah popup img di pano1 → key = pop-pano1-img-002

Tambah 3 popup di pano2 (gal, pdf, video) → keys berurutan

Live Preview (Mode A)

Mengubah field langsung terlihat

Toggle Preview | HTML berfungsi

Tombol Mobile/Tablet/Desktop/Full/Rotate mengubah ukuran iframe

Build Draft (Mode B)

Klik Build Draft pada pop-pano1-text-001

Buka tab Production → konten match (tanpa CDN)

Link CSS dist termuat

Build One & All

Build One: file berikut ada:

public/dist/core.css (pertama kali)

public/dist/popup-<key>.css (jika aktif per-popup)

public/inject/<key>.html

public/inject/registry.json ter-update (group by scene)

Build All: loop semua status=ready; registry.json.version ++

Loader

Di tur 3DVista, panggil openPopup('pop-pano1-text-001')

Iframe tampil; ESC/backdrop menutup; fokus kembali ke trigger

Export ZIP

Unduh tour-assets.zip

Isi: inject/*, dist/*, assets/*

Buka inject/*.html dalam ZIP → URL media mengarah ke assets/... dan tampil

Skala

Uji dengan 60+ popup → waktu Build All wajar, ukuran CSS per popup kecil (jika pipeline per-popup aktif)

11) Potongan Implementasi Kritis (ringkas)
11.1 Key Generator (service)
final class KeyFactory {
  public static function make(?Scene $scene, string $type): string {
    $slug = $scene?->slug ?: 'global';
    $prefix = "pop-{$slug}-{$type}-";
    $last = Popup::query()
      ->when($scene, fn($q)=>$q->where('scene_id',$scene->id), fn($q)=>$q->whereNull('scene_id'))
      ->where('type',$type)
      ->where('key','like',$prefix.'%')
      ->orderByDesc('key')
      ->value('key');
    $n = $last ? (int) substr($last,-3) + 1 : 1;
    return $prefix . str_pad((string)$n,3,'0',STR_PAD_LEFT);
  }
}

11.2 Registry Builder (group by scene)
$data = ['version'=>$newVersion, 'scenes'=>[]];
foreach (Popup::where('status','ready')->get() as $p) {
  $scene = optional($p->scene)->slug ?: 'global';
  $data['scenes'][$scene][$p->key] = [
    'type'=>$p->type,
    'file'=>"inject/{$p->key}.html",
    'preset'=>'wide',
  ];
}
File::put(public_path('inject/registry.json'), json_encode($data, JSON_PRETTY_PRINT|JSON_UNESCAPED_SLASHES));

11.3 Export ZIP – collect & rewrite assets (intinya)
// Kumpulkan URL lokal dari props_json:
$localPaths = CollectMedia::fromAllReadyPopups(); // return array of ['public'=> '/storage/...', 'abs'=> '/var/...']
CopyTo::dir($localPaths, public_path('assets'));
HtmlRewriter::rewriteStorageUrlsToAssets(public_path('inject')); // cari & ganti /storage/... -> assets/...
ZipBuilder::make(public_path('tour-assets.zip'))
  ->addDir(public_path('inject'),'inject')
  ->addDir(public_path('dist'),'dist')
  ->addDir(public_path('assets'),'assets')
  ->close();

11.4 Preview Ala HyperUI (kerangka)

Satu blade preview resources/views/filament/preview/popup.blade.php

Terima type, props dari state form; switch untuk merakit contentHtml.

Buat srcdoc template (Tailwind CDN).

Tombol: Preview|HTML + device size + rotate (semua pakai komponen Filament → auto tema).

12) Roadmap Opsional (bikin makin enak)

Preset ukuran per popup di registry (preset: 'sheet'|'modal'|'wide') → loader atur dimensi panel

Draft autosave & indikator “Perubahan belum di-build”

Bulk actions di Popups index: set status ready, Build massal

Queue build (jobs) untuk server low-spec

Audit log setiap Build/Export

13) Master Prompt (1 kalimat — siap tembak ke AI dev)

“Implement Mode A preview ala HyperUI (iframe srcdoc + toggle Preview|HTML + tombol Mobile/Tablet/Desktop/Full/Rotate) dan Mode B Production Preview (Build Draft + route /admin/preview/{key}), lengkapi actions Build One/All/Export ZIP (ikut assets & rewrite URL), pakai skema Scene→Popup dengan naming pop-<scene>-<type>-<nnn>, build inject/dist/registry (group by scene), upload file via FileUpload (atau URL), dan pastikan seluruh acceptance di atas lulus di server aaPanel.”

Kalau kamu mau, aku bisa kirim stub tambahan khusus arsitektur Scene/Popup (migrations, Resources + RelationManager, preview blade “HyperUI-like” dengan toggle, service build versi group by scene, export dengan rewrite assets). Tinggal bilang “drop stub scene+popup”, nanti aku tempelkan lengkap.

You said:
ok
ChatGPT said:
Thought for 14s

mantap. aku langsung drop paket stub Scene + Popup (siap tempel). Sudah include:

migrations (scenes, popups)

models

Filament: ScenesResource (+ PopupsRelationManager), PopupsResource (form dinamis per tipe)

KeyFactory (penamaan pop-<scene>-<type>-<nnn>)

PopupBuildService (build inject/dist/registry group by scene, Build Draft, Export ZIP + rewrite assets)

Blade Preview ala HyperUI (toggle Preview | HTML + Mobile/Tablet/Desktop/Full/Rotate)

Blade produksi (switch per tipe)

loader.js (+ helper by scene)

Tailwind configs v4 (core + per-popup)

route /admin/preview/{key}

Catatan: warna/komponen UI ikut tema Hasnayeen karena semua chrome pakai komponen Filament; di Blade preview aku tidak hard-code warna.

1) Database

database/migrations/2025_01_01_000000_create_scenes_table.php

<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
  public function up(): void {
    Schema::create('scenes', function (Blueprint $t) {
      $t->id();
      $t->string('slug')->unique(); // ex: pano1
      $t->string('title')->nullable();
      $t->unsignedInteger('order')->default(0);
      $t->timestamps();
    });
  }
  public function down(): void {
    Schema::dropIfExists('scenes');
  }
};


database/migrations/2025_01_01_000100_create_popups_table.php

<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
  public function up(): void {
    Schema::create('popups', function (Blueprint $t) {
      $t->id();
      $t->foreignId('scene_id')->nullable()->constrained()->cascadeOnDelete();
      $t->string('key')->unique(); // pop-<scene>-<type>-<nnn>
      $t->enum('type', ['text','img','gal','pdf','yt','video','audio','html','custom']);
      $t->enum('status', ['draft','ready'])->default('draft');
      $t->json('props_json')->nullable();
      $t->unsignedInteger('order')->default(0);
      $t->timestamps();
    });
  }
  public function down(): void {
    Schema::dropIfExists('popups');
  }
};

2) Models

app/Models/Scene.php

<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Scene extends Model
{
  protected $fillable = ['slug','title','order'];
  public function popups(): HasMany { return $this->hasMany(Popup::class); }
}


app/Models/Popup.php

<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Popup extends Model
{
  protected $fillable = ['scene_id','key','type','status','props_json','order'];
  protected $casts = ['props_json' => 'array'];
  public function scene(): BelongsTo { return $this->belongsTo(Scene::class); }
}

3) Key generator

app/Support/KeyFactory.php

<?php

namespace App\Support;

use App\Models\Popup;
use App\Models\Scene;

final class KeyFactory
{
  public static function make(?Scene $scene, string $type): string
  {
    $slug = $scene?->slug ?: 'global';
    $prefix = "pop-{$slug}-{$type}-";
    $last = Popup::query()
      ->when($scene, fn($q)=>$q->where('scene_id', $scene->id), fn($q)=>$q->whereNull('scene_id'))
      ->where('type',$type)
      ->where('key','like',$prefix.'%')
      ->orderByDesc('key')
      ->value('key');

    $n = $last ? (int)substr($last,-3) + 1 : 1;
    return $prefix . str_pad((string)$n, 3, '0', STR_PAD_LEFT);
  }
}

4) Filament Resources
4.1 Scenes

app/Filament/Resources/ScenesResource.php

<?php

namespace App\Filament\Resources;

use App\Filament\Resources\ScenesResource\Pages;
use App\Filament\Resources\ScenesResource\RelationManagers\PopupsRelationManager;
use App\Models\Scene;
use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Tables\Table;

class ScenesResource extends Resource
{
  protected static ?string $model = Scene::class;
  protected static ?string $navigationIcon = 'heroicon-o-photo';
  protected static ?string $navigationLabel = 'Scenes';

  public static function form(Form $form): Form
  {
    return $form->schema([
      Forms\Components\TextInput::make('slug')->required()->unique(ignoreRecord: true)->helperText('ex: pano1'),
      Forms\Components\TextInput::make('title')->label('Title'),
      Forms\Components\TextInput::make('order')->numeric()->default(0),
    ])->columns(2);
  }

  public static function table(Table $table): Table
  {
    return $table->columns([
      Tables\Columns\TextColumn::make('slug')->searchable(),
      Tables\Columns\TextColumn::make('title')->limit(40),
      Tables\Columns\TextColumn::make('popups_count')->counts('popups')->label('#Popups'),
      Tables\Columns\TextColumn::make('order'),
      Tables\Columns\TextColumn::make('updated_at')->since(),
    ])->actions([
      Tables\Actions\EditAction::make(),
    ])->defaultSort('order');
  }

  public static function getRelations(): array
  {
    return [ PopupsRelationManager::class ];
  }

  public static function getPages(): array
  {
    return [
      'index' => Pages\ListScenes::route('/'),
      'create' => Pages\CreateScene::route('/create'),
      'edit' => Pages\EditScene::route('/{record}/edit'),
    ];
  }
}


app/Filament/Resources/ScenesResource/RelationManagers/PopupsRelationManager.php

<?php

namespace App\Filament\Resources\ScenesResource\RelationManagers;

use App\Models\Popup;
use Filament\Forms;
use Filament\Resources\RelationManagers\RelationManager;
use Filament\Tables;

class PopupsRelationManager extends RelationManager
{
  protected static string $relationship = 'popups';
  protected static ?string $recordTitleAttribute = 'key';

  public function table(Tables\Table $table): Tables\Table
  {
    return $table->columns([
      Tables\Columns\TextColumn::make('key')->searchable(),
      Tables\Columns\TextColumn::make('type')->badge(),
      Tables\Columns\TextColumn::make('status')->badge(),
      Tables\Columns\TextColumn::make('order'),
      Tables\Columns\TextColumn::make('updated_at')->since(),
    ])->headerActions([
      Tables\Actions\CreateAction::make()
        ->using(function(array $data, $record){
          // handled in Popup Resource page
          return Popup::create($data);
        })
        ->url(fn() => route('filament.admin.resources.popups.create', ['scene' => $this->getOwnerRecord()->getKey()] ))
        ->openUrlInNewTab(),
    ])->actions([
      Tables\Actions\EditAction::make()->url(fn($r)=>route('filament.admin.resources.popups.edit',$r))->openUrlInNewTab(),
    ]);
  }
}


Pages untuk Scenes (generated): ListScenes, CreateScene, EditScene.

4.2 Popups (Editor + Preview ala HyperUI)

app/Filament/Resources/PopupsResource.php

<?php

namespace App\Filament\Resources;

use App\Filament\Resources\PopupsResource\Pages;
use App\Models\Popup;
use App\Models\Scene;
use App\Services\PopupBuildService;
use App\Support\KeyFactory;
use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Tables\Table;
use Illuminate\Support\Facades\Storage;

class PopupsResource extends Resource
{
  protected static ?string $model = Popup::class;
  protected static ?string $navigationIcon = 'heroicon-o-rectangle-stack';
  protected static ?string $navigationLabel = 'Popups';

  public static function form(Form $form): Form
  {
    return $form->schema([

      // === META ===
      Forms\Components\Section::make('Meta')->schema([
        Forms\Components\Select::make('scene_id')
          ->label('Scene')->options(Scene::query()->orderBy('order')->pluck('slug','id'))->searchable()
          ->helperText('Kosongkan untuk "global" popup'),
        Forms\Components\Select::make('type')->required()
          ->options(['text'=>'text','img'=>'img','gal'=>'gal','pdf'=>'pdf','yt'=>'yt','video'=>'video','audio'=>'audio','html'=>'html','custom'=>'custom'])
          ->reactive(),
        Forms\Components\Select::make('status')->options(['draft'=>'draft','ready'=>'ready'])->default('draft'),
        Forms\Components\TextInput::make('key')->disabled()->helperText('auto-generated: pop-<scene>-<type>-<nnn>'),
        Forms\Components\TextInput::make('order')->numeric()->default(0),
      ])->columns(2),

      // === PROPS (dinamis per type) ===
      Forms\Components\Section::make('Content')->schema([
        Forms\Components\Group::make()
          ->statePath('props_json')
          ->schema(self::propsSchema())
      ])->columns(1),

      // === PREVIEW ala HyperUI ===
      Forms\Components\Section::make('Preview')->schema([
        Forms\Components\ViewField::make('live_preview')
          ->view('filament.preview.popup') // lihat file blade di bawah
          ->reactive()
          ->columnSpanFull(),
      ])->collapsible(),

    ])->columns(2);
  }

  /** Dinamis form schema untuk props_json */
  private static function propsSchema(): array
  {
    // Toggle sumber (Upload vs URL) untuk beberapa tipe
    $urlToggle = fn($name) => Forms\Components\Toggle::make("use_url_$name")->inline(false)->default(false);

    return [
      Forms\Components\Fieldset::make('text')->visible(fn($get)=>$get('../../type')==='text')->schema([
        Forms\Components\TextInput::make('title'),
        Forms\Components\Textarea::make('body')->rows(4),
      ]),
      Forms\Components\Fieldset::make('img')->visible(fn($get)=>$get('../../type')==='img')->schema([
        Forms\Components\Toggle::make('use_url')->label('Gunakan URL?')->default(false)->reactive(),
        Forms\Components\FileUpload::make('src')->image()->directory('uploads/images')
          ->visible(fn($get)=>!$get('use_url')),
        Forms\Components\TextInput::make('src')->label('Image URL')->url()
          ->visible(fn($get)=>$get('use_url')),
        Forms\Components\TextInput::make('caption'),
      ]),
      Forms\Components\Fieldset::make('gal')->visible(fn($get)=>$get('../../type')==='gal')->schema([
        Forms\Components\Repeater::make('items')->schema([
          Forms\Components\Toggle::make('use_url')->label('URL?')->default(false)->reactive(),
          Forms\Components\FileUpload::make('src')->image()->directory('uploads/images')->visible(fn($get)=>!$get('use_url')),
          Forms\Components\TextInput::make('src')->label('Image URL')->url()->visible(fn($get)=>$get('use_url')),
          Forms\Components\TextInput::make('caption'),
        ])->reorderable()->minItems(1),
      ]),
      Forms\Components\Fieldset::make('pdf')->visible(fn($get)=>$get('../../type')==='pdf')->schema([
        Forms\Components\Toggle::make('use_url')->label('URL?')->default(false)->reactive(),
        Forms\Components\FileUpload::make('src')->directory('uploads/docs')->acceptedFileTypes(['application/pdf'])->visible(fn($get)=>!$get('use_url')),
        Forms\Components\TextInput::make('src')->label('PDF URL')->url()->visible(fn($get)=>$get('use_url')),
      ]),
      Forms\Components\Fieldset::make('video')->visible(fn($get)=>$get('../../type')==='video')->schema([
        Forms\Components\Toggle::make('use_url')->label('URL?')->default(true)->reactive(),
        Forms\Components\FileUpload::make('src')->directory('uploads/videos')->acceptedFileTypes(['video/mp4','video/webm'])->visible(fn($get)=>!$get('use_url')),
        Forms\Components\TextInput::make('src')->label('Video URL')->url()->visible(fn($get)=>$get('use_url')),
        Forms\Components\Toggle::make('poster_url')->label('Poster via URL?')->default(true)->reactive(),
        Forms\Components\FileUpload::make('poster')->image()->directory('uploads/images')->visible(fn($get)=>!$get('poster_url')),
        Forms\Components\TextInput::make('poster')->label('Poster URL')->url()->visible(fn($get)=>$get('poster_url')),
      ]),
      Forms\Components\Fieldset::make('audio')->visible(fn($get)=>$get('../../type')==='audio')->schema([
        Forms\Components\Toggle::make('use_url')->label('URL?')->default(true)->reactive(),
        Forms\Components\FileUpload::make('src')->directory('uploads/audios')->acceptedFileTypes(['audio/mpeg','audio/ogg'])->visible(fn($get)=>!$get('use_url')),
        Forms\Components\TextInput::make('src')->label('Audio URL')->url()->visible(fn($get)=>$get('use_url')),
        Forms\Components\TextInput::make('title')->label('Title (opsional)'),
      ]),
      Forms\Components\Fieldset::make('yt')->visible(fn($get)=>$get('../../type')==='yt')->schema([
        Forms\Components\TextInput::make('videoId')->label('YouTube Video ID / URL'),
      ]),
      Forms\Components\Fieldset::make('html')->visible(fn($get)=>$get('../../type')==='html')->schema([
        Forms\Components\Textarea::make('html')->rows(8)->helperText('Konten HTML (akan disanitasi saat render produksi)'),
      ]),
      Forms\Components\Fieldset::make('custom')->visible(fn($get)=>$get('../../type')==='custom')->schema([
        Forms\Components\Textarea::make('json')->rows(8)->helperText('JSON bebas untuk komponen kustom'),
      ]),
    ];
  }

  public static function table(Table $table): Table
  {
    return $table->columns([
      Tables\Columns\TextColumn::make('key')->searchable()->copyable(),
      Tables\Columns\TextColumn::make('scene.slug')->label('Scene')->badge(),
      Tables\Columns\TextColumn::make('type')->badge(),
      Tables\Columns\TextColumn::make('status')->badge(),
      Tables\Columns\TextColumn::make('order'),
      Tables\Columns\TextColumn::make('updated_at')->since(),
    ])->filters([
      Tables\Filters\SelectFilter::make('scene')->relationship('scene','slug'),
      Tables\Filters\SelectFilter::make('type')->options(array_combine(
        ['text','img','gal','pdf','yt','video','audio','html','custom'],
        ['text','img','gal','pdf','yt','video','audio','html','custom']
      )),
      Tables\Filters\SelectFilter::make('status')->options(['draft'=>'draft','ready'=>'ready']),
    ])->actions([
      Tables\Actions\EditAction::make(),
      Tables\Actions\Action::make('Build Draft')->color('warning')->action(fn($r)=>app(PopupBuildService::class)->buildDraft($r)),
      Tables\Actions\Action::make('Build')->color('success')->action(fn($r)=>app(PopupBuildService::class)->buildOne($r)),
    ])->headerActions([
      Tables\Actions\Action::make('Build All')->color('primary')->action(fn()=>app(PopupBuildService::class)->buildAll()),
      Tables\Actions\Action::make('Export ZIP')->color('gray')->action(fn()=>app(PopupBuildService::class)->exportZip()),
    ])->defaultSort('updated_at','desc');
  }

  public static function getPages(): array
  {
    return [
      'index' => Pages\ListPopups::route('/'),
      'create' => Pages\CreatePopup::route('/create'),
      'edit' => Pages\EditPopup::route('/{record}/edit'),
    ];
  }
}


Hook penamaan otomatis (di CreatePopup page)

app/Filament/Resources/PopupsResource/Pages/CreatePopup.php

<?php

namespace App\Filament\Resources\PopupsResource\Pages;

use App\Filament\Resources\PopupsResource;
use App\Models\Scene;
use App\Support\KeyFactory;
use Filament\Resources\Pages\CreateRecord;

class CreatePopup extends CreateRecord
{
  protected static string $resource = PopupsResource::class;

  protected function mutateFormDataBeforeCreate(array $data): array
  {
    $scene = isset($data['scene_id']) ? Scene::find($data['scene_id']) : null;
    $data['key'] = KeyFactory::make($scene, $data['type']);
    // Resolve FileUpload -> URL jika ada file path
    $data['props_json'] = $this->normalizeProps($data['type'], $data['props_json'] ?? []);
    return $data;
  }

  private function normalizeProps(string $type, array $props): array
  {
    $toUrl = function($val) {
      if (is_string($val)) return $val;
      if (is_array($val) && isset($val['path'])) return \Storage::url($val['path']);
      return $val;
    };

    switch ($type) {
      case 'img':
        $props['src'] = $toUrl($props['src'] ?? '');
        break;
      case 'gal':
        $props['items'] = array_map(function($it) use ($toUrl) {
          $it['src'] = $toUrl($it['src'] ?? '');
          return $it;
        }, $props['items'] ?? []);
        break;
      case 'pdf':
      case 'video':
      case 'audio':
        $props['src'] = $toUrl($props['src'] ?? '');
        if ($type==='video' && !empty($props['poster'])) $props['poster'] = $toUrl($props['poster']);
        break;
    }
    return $props;
  }
}


EditPopup: lakukan normalisasi serupa di mutateFormDataBeforeSave.

5) Preview ala HyperUI (Mode A)

resources/views/filament/preview/popup.blade.php

@php
/** @var \Filament\Forms\Contracts\HasForms $this */
$state = $this->form->getState() ?? [];
$type  = $state['type'] ?? 'text';
$key   = $state['key'] ?? '(auto)';
$props = $state['props_json'] ?? [];
// builder konten:
function h($s){ return e($s ?? ''); }
function buildContent($type,$p){
  switch($type){
    case 'text':
      $t = h($p['title'] ?? 'Title'); $b = h($p['body'] ?? 'Body');
      return "<section class=\"mx-auto max-w-2xl p-6\"><h2 class=\"text-2xl font-semibold\">$t</h2><p class=\"mt-3 text-gray-600\">$b</p></section>";
    case 'img':
      $src = h($p['src'] ?? ''); $cap = h($p['caption'] ?? '');
      return "<div class=\"mx-auto max-w-3xl p-6\"><img class=\"w-full rounded-lg object-cover\" src=\"$src\" alt=\"\"/><p class=\"mt-2 text-sm text-gray-500\">$cap</p></div>";
    case 'gal':
      $items = $p['items'] ?? [];
      $cards = '';
      foreach($items as $it){ $s=h($it['src']??''); $c=h($it['caption']??'');
        $cards .= "<figure class=\"relative\"><img class=\"w-full rounded-lg object-cover\" src=\"$s\" alt=\"\"/><figcaption class=\"mt-1 text-xs text-gray-500\">$c</figcaption></figure>";
      }
      return "<div class=\"mx-auto max-w-6xl p-6 grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6\">$cards</div>";
    case 'pdf':
      $src = h($p['src'] ?? '');
      return "<div class=\"p-6\"><iframe class=\"w-full h-[560px] rounded-lg border\" src=\"$src\"></iframe></div>";
    case 'video':
      $src = h($p['src'] ?? ''); $poster = h($p['poster'] ?? '');
      return "<div class=\"p-6\"><video class=\"w-full rounded-lg\" controls ".($poster?"poster=\"$poster\"":"")."><source src=\"$src\"></video></div>";
    case 'audio':
      $src = h($p['src'] ?? '');
      return "<div class=\"p-6\"><audio class=\"w-full\" controls src=\"$src\"></audio></div>";
    case 'yt':
      $v = h($p['videoId'] ?? '');
      if (str_contains($v,'http')) {
        // quick parse id
        preg_match('/v=([^&]+)/',$v,$m); $v=$m[1]??$v;
      }
      return "<div class=\"p-6\"><iframe class=\"w-full aspect-video rounded-lg\" src=\"https://www.youtube.com/embed/$v\" allowfullscreen></iframe></div>";
    case 'html':
      // aman di Mode A (iframe sandbox), untuk Code view tetap escape
      return $p['html'] ?? '';
    default:
      return "<div class=\"p-6 text-sm text-gray-500\">Custom / Unknown type</div>";
  }
}
$content = buildContent($type,$props);

// template srcdoc (Tailwind CDN)
$html = <<<HTML
<!doctype html><html><head>
<meta charset="utf-8"/><meta name="viewport" content="width=device-width,initial-scale=1"/>
<script src="https://cdn.tailwindcss.com"></script>
</head><body class="bg-white">$content</body></html>
HTML;

// versi untuk tab "HTML"
$showHtml = htmlspecialchars($html, ENT_QUOTES);
$srcdoc   = htmlspecialchars($html, ENT_QUOTES);
@endphp

<div class="fi-section"> {{-- biarkan theme Hasnayeen yang atur warna --}}
  <div class="flex items-center justify-between mb-2">
    <div class="text-xs fi-section-header-heading">HyperUI-like Preview</div>
    <div class="flex items-center gap-2">
      <button type="button" class="fi-btn" onclick="pvSetW(375)">Mobile</button>
      <button type="button" class="fi-btn" onclick="pvSetW(768)">Tablet</button>
      <button type="button" class="fi-btn" onclick="pvSetW(1280)">Desktop</button>
      <button type="button" class="fi-btn" onclick="pvSetW(99999)">Full</button>
      <button type="button" class="fi-btn" onclick="pvRotate()">Rotate</button>
      <span class="mx-2"></span>
      <button type="button" class="fi-btn" onclick="pvMode('preview')">Preview</button>
      <button type="button" class="fi-btn" onclick="pvMode('code')">HTML</button>
    </div>
  </div>

  <div id="pv-wrap" class="rounded-xl border">
    <iframe id="pv" class="w-full" style="width:1280px;height:640px" srcdoc="{{ $srcdoc }}" title="preview"></iframe>
    <pre id="pv-code" class="hidden p-4 text-xs overflow-auto"><code>{{ $showHtml }}</code></pre>
  </div>

  <div class="mt-2 text-[11px] opacity-70">Key: <code>{{ $key }}</code> · Type: <code>{{ $type }}</code></div>
</div>

<script>
(function(){
  const wrap = document.currentScript.previousElementSibling;
  const pv   = wrap.querySelector('#pv');
  const code = wrap.querySelector('#pv-code');
  window.pvSetW = (w)=> pv.style.width = (w===99999? '100%': (w+'px'));
  window.pvRotate = ()=>{ const w=pv.offsetWidth,h=pv.offsetHeight; pv.style.width=h+'px'; pv.style.height=w+'px'; };
  window.pvMode = (m)=>{ if(m==='code'){ pv.classList.add('hidden'); code.classList.remove('hidden'); } else { code.classList.add('hidden'); pv.classList.remove('hidden'); } };
})();
</script>

6) Blade produksi (switch per tipe)

resources/views/popup/render.blade.php

@php
/** @var \App\Models\Popup $popup */
$type = $popup->type;
$p    = $popup->props_json ?? [];
function h($s){ return e($s ?? ''); }
@endphp
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8"/><meta name="viewport" content="width=device-width,initial-scale=1"/>
  <link rel="stylesheet" href="../dist/core.css">
  {{-- jika aktif CSS per popup --}}
  @if (file_exists(public_path("dist/popup-{$popup->key}.css")))
    <link rel="stylesheet" href="../dist/popup-{{ $popup->key }}.css">
  @endif
  <title>{{ $popup->key }}</title>
</head>
<body class="bg-white">
  @switch($type)
    @case('text')
      <section class="mx-auto max-w-2xl p-6">
        <h2 class="text-2xl font-semibold">{{ h($p['title'] ?? '') }}</h2>
        <p class="mt-3 text-gray-600">{{ h($p['body'] ?? '') }}</p>
      </section>
      @break
    @case('img')
      <div class="mx-auto max-w-3xl p-6">
        <img class="w-full rounded-lg object-cover" src="{{ $p['src'] ?? '' }}" alt="">
        @if(!empty($p['caption']))<p class="mt-2 text-sm text-gray-500">{{ h($p['caption']) }}</p>@endif
      </div>
      @break
    @case('gal')
      @php $items = $p['items'] ?? []; @endphp
      <div class="mx-auto max-w-6xl p-6 grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
        @foreach($items as $it)
          <figure>
            <img class="w-full rounded-lg object-cover" src="{{ $it['src'] ?? '' }}" alt="">
            @if(!empty($it['caption']))<figcaption class="mt-1 text-xs text-gray-500">{{ h($it['caption']) }}</figcaption>@endif
          </figure>
        @endforeach
      </div>
      @break
    @case('pdf')
      <div class="p-6"><iframe class="w-full h-[560px] rounded-lg border" src="{{ $p['src'] ?? '' }}"></iframe></div>
      @break
    @case('video')
      <div class="p-6"><video class="w-full rounded-lg" controls @if(!empty($p['poster'])) poster="{{ $p['poster'] }}" @endif><source src="{{ $p['src'] ?? '' }}"></video></div>
      @break
    @case('audio')
      <div class="p-6"><audio class="w-full" controls src="{{ $p['src'] ?? '' }}"></audio></div>
      @break
    @case('yt')
      @php $v = $p['videoId'] ?? ''; if (str_contains($v,'http')) { preg_match('/v=([^&]+)/',$v,$m); $v=$m[1]??$v; } @endphp
      <div class="p-6"><iframe class="w-full aspect-video rounded-lg" src="https://www.youtube.com/embed/{{ $v }}" allowfullscreen></iframe></div>
      @break
    @case('html')
      {{-- sanitize server-side jika diperlukan --}}
      {!! $p['html'] ?? '' !!}
      @break
    @default
      <div class="p-6 text-sm text-gray-500">Unknown type</div>
  @endswitch

  <div class="p-6">
    <button onclick="window.parent?.closePopup?.()" class="btn">Close ✕</button>
  </div>
</body>
</html>

7) Service build (group by scene + export assets)

app/Services/PopupBuildService.php

<?php

namespace App\Services;

use App\Models\Popup;
use Illuminate\Support\Facades\File;
use Illuminate\Support\Str;
use Symfony\Component\Process\Process;
use ZipArchive;

class PopupBuildService
{
  private string $publicPath;
  private string $injectDir;
  private string $distDir;
  private string $previewDir;
  private string $assetsDir;

  public function __construct()
  {
    $this->publicPath = public_path();
    $this->injectDir  = $this->publicPath.'/inject';
    $this->distDir    = $this->publicPath.'/dist';
    $this->assetsDir  = $this->publicPath.'/assets';
    $this->previewDir = storage_path('app/preview');
    File::ensureDirectoryExists($this->injectDir);
    File::ensureDirectoryExists($this->distDir);
    File::ensureDirectoryExists($this->assetsDir);
    File::ensureDirectoryExists($this->previewDir);
  }

  public function buildAll(): void
  {
    $this->buildCoreCss();
    foreach (Popup::query()->where('status','ready')->get() as $p) {
      $this->buildOne($p, buildCore:false);
    }
    $this->writeRegistry(versionBump:true);
  }

  public function buildOne(Popup $popup, bool $buildCore = true): void
  {
    if ($buildCore) $this->buildCoreCss();

    // Render sementara untuk purge per-popup css (opsional)
    $temp = $this->previewDir.'/tmp-'.$popup->key.'.html';
    File::put($temp, view('popup.render', ['popup'=>$popup])->render());
    $this->buildPopupCss($popup->key, $temp); // opsional

    // Render final ke inject
    $htmlPath = $this->injectDir.'/'.$popup->key.'.html';
    File::put($htmlPath, view('popup.render', ['popup'=>$popup])->render());

    // Registry
    $this->writeRegistry();
    @unlink($temp);
  }

  public function buildDraft(Popup $popup): string
  {
    $this->buildCoreCss();
    $temp = $this->previewDir.'/draft-'.$popup->key.'.html';
    $this->buildPopupCss($popup->key, $temp);
    File::put($temp, view('popup.render', ['popup'=>$popup])->render());
    return $temp;
  }

  public function exportZip(): string
  {
    // 1) Kumpulkan semua media lokal dari props_json
    $paths = $this->collectLocalMediaPaths();
    // 2) Copy ke public/assets (mirror)
    foreach ($paths as $abs => $rel) {
      $dest = $this->assetsDir.'/'.$rel;
      File::ensureDirectoryExists(dirname($dest));
      if (File::exists($abs)) File::copy($abs, $dest);
    }
    // 3) Rewrite URL /storage/... -> assets/... pada inject html (sementara copy ke folder tmp rewrite)
    $this->rewriteStorageUrlsToAssets($this->injectDir);

    // 4) Bundle
    $zipPath = $this->publicPath.'/tour-assets.zip';
    @unlink($zipPath);
    $zip = new ZipArchive;
    $zip->open($zipPath, ZipArchive::CREATE);
    $this->zipDir($this->injectDir, $zip, 'inject');
    $this->zipDir($this->distDir, $zip, 'dist');
    $this->zipDir($this->assetsDir, $zip, 'assets');
    $zip->close();

    // Optional: balikin inject HTML ke /storage jika mau (skip: hasil rewrite aman juga untuk runtime local)
    return $zipPath;
  }

  private function collectLocalMediaPaths(): array
  {
    $map = [];
    $add = function(?string $url) use (&$map) {
      if (!$url || !Str::startsWith($url, '/storage/')) return;
      $abs = public_path(ltrim($url,'/'));
      $rel = ltrim(str_replace('/storage/','',$url),'/'); // assets/<rel>
      $map[$abs] = $rel;
    };

    foreach (Popup::where('status','ready')->get() as $p) {
      $props = $p->props_json ?? [];
      switch ($p->type) {
        case 'img':   $add($props['src'] ?? null); break;
        case 'gal':   foreach (($props['items'] ?? []) as $it) $add($it['src'] ?? null); break;
        case 'pdf':
        case 'video':
        case 'audio': $add($props['src'] ?? null); if ($p->type==='video' && !empty($props['poster'])) $add($props['poster']); break;
      }
    }
    return $map;
  }

  private function rewriteStorageUrlsToAssets(string $dir): void
  {
    $it = new \RecursiveIteratorIterator(new \RecursiveDirectoryIterator($dir));
    foreach ($it as $f) {
      if ($f->isDir()) continue;
      if (pathinfo($f->getFilename(), PATHINFO_EXTENSION) !== 'html') continue;
      $html = File::get($f->getRealPath());
      $html = str_replace('src="/storage/', 'src="../assets/', $html);
      $html = str_replace('poster="/storage/', 'poster="../assets/', $html);
      File::put($f->getRealPath(), $html);
    }
  }

  private function writeRegistry(bool $versionBump=false): void
  {
    $path = $this->injectDir.'/registry.json';
    $data = File::exists($path) ? json_decode(File::get($path), true) : ['version'=>1,'scenes'=>[]];
    if ($versionBump) $data['version'] = ($data['version'] ?? 1) + 1;

    $data['scenes'] = [];
    foreach (Popup::where('status','ready')->get() as $p) {
      $scene = optional($p->scene)->slug ?: 'global';
      $data['scenes'][$scene][$p->key] = [
        'type' => $p->type,
        'file' => "inject/{$p->key}.html",
        'preset' => 'wide',
      ];
    }
    File::put($path, json_encode($data, JSON_PRETTY_PRINT|JSON_UNESCAPED_SLASHES));
  }

  private function buildCoreCss(): void
  {
    $cmd = [$this->npx(),'tailwindcss','-c','tailwind.core.cjs','-i','resources/tour/core.css','-o',$this->distDir.'/core.css','--minify'];
    $this->run($cmd);
  }

  private function buildPopupCss(string $key, string $contentFile): void
  {
    if (!File::exists($contentFile)) File::put($contentFile,'<!doctype html><html><body></body></html>');
    $out = $this->distDir.'/popup-'.$key.'.css';
    $cmd = [$this->npx(),'tailwindcss','-c','tailwind.popup.cjs','-i','resources/tour/popup.css','-o',$out,'--minify','--content',$contentFile];
    $this->run($cmd);
  }

  private function zipDir(string $dir, ZipArchive $zip, string $prefix=''): void
  {
    if (!is_dir($dir)) return;
    $files = new \RecursiveIteratorIterator(new \RecursiveDirectoryIterator($dir));
    foreach ($files as $f) {
      if ($f->isDir()) continue;
      $filePath = $f->getRealPath();
      $local = trim($prefix.'/'.ltrim(str_replace($dir,'',$filePath),'/'),'/');
      $zip->addFile($filePath, $local);
    }
  }

  private function run(array $cmd): void
  {
    $p = new Process($cmd, base_path());
    $p->setTimeout(300); $p->run();
    if (!$p->isSuccessful()) throw new \RuntimeException($p->getErrorOutput() ?: $p->getOutput());
  }

  private function npx(): string
  {
    // return '/usr/local/node/bin/npx'; // kalau perlu path absolut di aaPanel
    return 'npx';
  }
}

8) Route Production Preview

routes/web.php

use Illuminate\Support\Facades\Route;

Route::get('/admin/preview/{key}', function(string $key){
  $path = storage_path('app/preview/draft-'.$key.'.html');
  abort_unless(file_exists($path), 404);
  return response(file_get_contents($path))
    ->header('Content-Type','text/html')
    ->header('Cache-Control','no-store, no-cache, must-revalidate, max-age=0');
})->middleware(['web','auth']);

9) Tailwind v4 config & sumber CSS

tailwind.core.cjs

module.exports = {
  content: [],
  safelist: [
    'mx-auto','max-w-2xl','max-w-3xl','max-w-6xl','p-6',
    'grid','grid-cols-1','sm:grid-cols-2','lg:grid-cols-3','gap-6',
    'text-2xl','font-semibold','text-gray-600','text-gray-500',
    'w-full','rounded-lg','object-cover','aspect-video','border','rounded-xl',
  ],
  theme: { extend: {} },
  plugins: [],
}


tailwind.popup.cjs

module.exports = {
  content: [], // diisi via --content file HTML sementara
  theme: { extend: {} },
  plugins: [],
}


resources/tour/core.css

@tailwind base;
@tailwind components;
@tailwind utilities;

.btn { @apply inline-flex items-center gap-2 rounded-lg border px-3 py-2 text-sm; }


resources/tour/popup.css

@tailwind base;
@tailwind components;
@tailwind utilities;

10) Loader + helper by scene

public/loader.js

(function(){
  const ID = 'inject-overlay';
  let prevFocus = null, REG = null, LOADED = false;

  async function registry(){
    if (LOADED) return REG;
    const r = await fetch('inject/registry.json', { cache: 'no-store' });
    REG = await r.json(); LOADED = true; return REG;
  }

  function esc(e){ if (e.key === 'Escape') closePopup(); }

  function buildOverlay(src){
    let host = document.getElementById(ID);
    if (!host) { host = document.createElement('div'); host.id = ID; document.body.appendChild(host); }
    host.innerHTML = `
<style>
#${ID}{position:fixed;inset:0;z-index:100000}
#${ID} .backdrop{position:absolute;inset:0;background:rgba(0,0,0,.6)}
#${ID} .panel{position:relative;margin:2rem auto;background:#fff;display:flex;flex-direction:column;overflow:hidden;transform:scale(.98);opacity:.0;transition:.2s;max-width:1100px;max-height:90vh;border-radius:1rem}
#${ID}.show .panel{transform:scale(1);opacity:1}
#${ID} iframe{width:100%;height:100%;border:0}
</style>
<div class="backdrop"></div>
<div class="panel"><iframe src="${src}"></iframe></div>`;
    const overlay = host;
    overlay.querySelector('.backdrop').onclick = closePopup;
    setTimeout(()=> overlay.classList.add('show'), 10);
    prevFocus = document.activeElement;
    overlay.querySelector('.panel').tabIndex = -1;
    overlay.querySelector('.panel').focus();
    document.addEventListener('keydown', esc);
  }

  window.openPopup = async function(key){
    const r = await registry();
    const map = r.map || null; // legacy
    const scenes = r.scenes || null;
    let file = null;

    if (map && map[key]) file = map[key].file;
    if (!file && scenes){
      for (const s of Object.keys(scenes)){
        if (key in scenes[s]) { file = scenes[s][key].file; break; }
      }
    }
    if (!file) return console.error('popup not found', key);

    buildOverlay(`${file}?v=${r.version||1}`);
  };

  window.closePopup = function(){
    const el = document.getElementById(ID); if (!el) return;
    el.classList.remove('show');
    setTimeout(()=>{ el.remove(); document.removeEventListener('keydown', esc); if (prevFocus && prevFocus.focus) prevFocus.focus(); }, 180);
  };

  // helpers
  window.listScene = async function(sceneSlug){
    const r = await registry(); return Object.keys((r.scenes||{})[sceneSlug] || {});
  };
  window.openScenePopup = async function(sceneSlug, type, index1){
    const keys = await window.listScene(sceneSlug);
    const match = keys.filter(k => k.startsWith(`pop-${sceneSlug}-${type}-`)).sort();
    const key = match[index1-1]; if (key) openPopup(key); else console.warn('no popup at index', index1);
  };
})();

11) Quick wiring

php artisan migrate

php artisan storage:link

Buat Scenes (pano1, pano2…)

Tambah Popup via Scene Relation → pilih type, isi props

Coba Preview (toggle Preview | HTML, device buttons)

Build Draft → tab Production (/admin/preview/{key})

Build → cek public/inject, public/dist, registry.json

Export ZIP → public/tour-assets.zip (inject/dist/assets)
