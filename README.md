# Membuat Templating View pada Codeigniter 3 seperti di Vue Js
## 🎯 FITUR YANG DIDUKUNG (CONFIRMED)
| Vue Feature     | CI3 Component |
| --------------- | ------------- |
| props           | ✅             |
| default props   | ✅             |
| boolean props   | ✅             |
| slot            | ✅             |
| v-bind attrs    | ✅             |
| @events         | ✅             |
| class binding   | ✅             |
| v-model (basic) | ✅             |
| emits           | ✅             |

---

## 🧠 ARSITEKTUR INTI
```
PHP (Server)
└── render component + props + slot
HTML (Contract)
└── data-* attributes
JavaScript (Runtime)
└── v-model + emits + events
```

Ini bukan fake Vue, tapi Vue-pattern yang realistis untuk CI3.

---

## 1. STRUKTUR FOLDER
```
application/
 ├── controllers/
 │   └── Example.php
 ├── helpers/
 │   └── component_helper.php
 ├── views/
 │   ├── layouts/
 │   │   └── app.php
 │   ├── pages/
 │   │   └── example.php
 │   └── components/
 │       ├── input.php
 │       └── card.php
assets/
 └── js/
     └── components.js
```

---

## 2. CORE COMPONENT HELPER (ENGINE)

📁 application/helpers/component_helper.php
```
<?php
defined('BASEPATH') OR exit('No direct script access allowed');

function component($name, $props = [], $slot = null)
{
    $CI =& get_instance();

    return $CI->load->view("components/{$name}", [
        'props' => $props,
        'slot'  => $slot
    ], TRUE);
}

/**
 * Vue-like v-bind attribute renderer
 */
function bind_attrs($attrs = [])
{
    $html = '';
    foreach ($attrs as $key => $value) {
        if ($value === true) {
            $html .= " {$key}";
        } elseif ($value !== false && $value !== null) {
            $html .= " {$key}=\"" . html_escape($value) . "\"";
        }
    }
    return $html;
}
```

📌 Autoload:
```
$autoload['helper'] = ['component'];
```

---

## 3. MAIN APP TEMPLATE (LAYOUT)

📁 application/views/layouts/app.php
```
<!DOCTYPE html>
<html>
<head>
    <title><?= $title ?? 'CI3 Vue-like Component' ?></title>
    <style>
        body { font-family: Arial; padding:40px }
        .card { border:1px solid #ddd; padding:16px; margin-bottom:16px }
        .form-control { width:100%; padding:8px }
        .mb-3 { margin-bottom:12px }
    </style>
</head>
<body>

<?= $content ?>

<script src="/assets/js/components.js"></script>
</body>
</html>
```

---

## 4 INPUT COMPONENT (FULL FEATURE)

📁 application/views/components/input.php
```
<?php
// DEFAULT PROPS (Vue style)
$defaults = [
    'type'       => 'text',
    'label'      => null,
    'name'       => null,
    'model'      => null,
    'modelValue' => '',
    'class'      => '',
    'required'   => false,
    'disabled'   => false,
    'emits'      => [],
];

// MERGE PROPS
$p = array_merge($defaults, $props);

// SPECIAL BINDINGS
$attrs  = $props[':attrs'] ?? [];
$events = $props['@events'] ?? [];

// EVENT ATTRIBUTES
$eventAttrs = [];
foreach ($events as $event => $handler) {
    $eventAttrs["on{$event}"] = "{$handler}(this)";
}

// EMITS ATTRIBUTE
$emitAttr = '';
if (!empty($p['emits'])) {
    $emitAttr = 'data-emits="' . implode(',', $p['emits']) . '"';
}
?>

<div class="form-group <?= html_escape($p['class']) ?>">
    <?php if ($p['label']): ?>
        <label><?= html_escape($p['label']) ?></label>
    <?php endif; ?>

    <input
        type="<?= $p['type'] ?>"
        name="<?= $p['name'] ?>"
        value="<?= html_escape($p['modelValue']) ?>"
        data-component="input"
        data-model="<?= html_escape($p['model']) ?>"
        <?= $emitAttr ?>
        class="form-control"
        <?= bind_attrs([
            'required' => $p['required'],
            'disabled' => $p['disabled'],
            ...$attrs,
            ...$eventAttrs
        ]) ?>
    >
</div>
```

---

## 5. COMPONENT TEXTAREA

📁 application/views/components/textarea.php
```
<?php
// DEFAULT PROPS (Vue-style)
$defaults = [
    'label'      => null,
    'name'       => null,
    'model'      => null,
    'modelValue' => '',
    'rows'       => 4,
    'class'      => '',
    'required'   => false,
    'disabled'   => false,
    'emits'      => [],
];

// MERGE PROPS
$p = array_merge($defaults, $props);

// SPECIAL BINDINGS
$attrs  = $props[':attrs'] ?? [];
$events = $props['@events'] ?? [];

// EVENT ATTRIBUTES
$eventAttrs = [];
foreach ($events as $event => $handler) {
    $eventAttrs["on{$event}"] = "{$handler}(this)";
}

// EMITS ATTRIBUTE
$emitAttr = '';
if (!empty($p['emits'])) {
    $emitAttr = 'data-emits="' . implode(',', $p['emits']) . '"';
}
?>

<div class="form-group <?= html_escape($p['class']) ?>">
    <?php if ($p['label']): ?>
        <label><?= html_escape($p['label']) ?></label>
    <?php endif; ?>

    <textarea
        name="<?= $p['name'] ?>"
        rows="<?= (int) $p['rows'] ?>"
        data-component="textarea"
        data-model="<?= html_escape($p['model']) ?>"
        <?= $emitAttr ?>
        class="form-control"
        <?= bind_attrs([
            'required' => $p['required'],
            'disabled' => $p['disabled'],
            ...$attrs,
            ...$eventAttrs
        ]) ?>
    ><?= html_escape($p['modelValue']) ?><?= $slot ?></textarea>
</div>
```

---

## 6. SLOT COMPONENT (CARD)

📁 application/views/components/card.php
```
<div class="card">
    <?php if (!empty($props['title'])): ?>
        <h3><?= html_escape($props['title']) ?></h3>
    <?php endif; ?>

    <?= $slot ?>
</div>
```

---

## 7. JAVASCRIPT RUNTIME (V-MODEL + EMITS)

📁 assets/js/components.js
```
const Store = {};

document.addEventListener('DOMContentLoaded', () => {

  document.querySelectorAll('[data-component]').forEach(el => {
    const model = el.dataset.model;
    const emits = el.dataset.emits?.split(',') || [];

    // INIT v-model
    if (model) {
      Store[model] = el.value;
    }

    el.addEventListener('input', () => {

      if (model) {
        Store[model] = el.value;
      }

      // EMITS
      emits.forEach(event => {
        document.dispatchEvent(
          new CustomEvent(event, {
            detail: {
              model,
              value: el.value
            }
          })
        );
      });

    });

  });

});
```

---

## 8. CONTROLLER (MAIN APP)

📁 application/controllers/Example.php
```
class Example extends CI_Controller
{
    public function index()
    {
        $data['content'] = $this->load->view('pages/example', [], TRUE);
        $this->load->view('layouts/app', $data);
    }
}
```

---

## 9. PAGE USAGE (CONSUMER SIDE)

📁 application/views/pages/example.php
```
<?= component('card', ['title' => 'Vue-like Input Component'], '

'.component('input', [
    "label" => "Email",
    "name" => "email",
    "model" => "email",
    "modelValue" => "",
    "required" => true,
    "class" => "mb-3",
    ":attrs" => [
        "placeholder" => "Masukkan email",
        "maxlength" => 100
    ],
    "@events" => [
        "blur" => "console.log"
    ],
    "emits" => ["update"]
]).'

') ?>

<script>
document.addEventListener('update', e => {
    console.log('EMIT update:', e.detail);
    console.log('v-model value:', Store[e.detail.model]);
});
</script>
```

---

## 10. HASIL AKHIR (REAL)

* Props & default props
* Boolean props (```required```, ```disabled```)
* Slot (```card```)
* v-bind attrs (```:attrs```)
* Event binding (```@events```)
* Class binding
* v-model (Store)
* Emits (CustomEvent)
