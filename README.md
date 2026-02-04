# Membuat Templating View pada Codeigniter 3 seperti di Vue Js
## 🎯 Target Akhir (Vue-like Usage)
```
<?= component('input', [
    'modelValue' => set_value('email'),
    'name'       => 'email',
    'label'      => 'Email',
    'type'       => 'email',
    'required'   => true,
    'disabled'   => false,
    'class'      => 'mb-3',
    ':attrs'     => [
        'placeholder' => 'Masukkan email',
        'maxlength'  => 100
    ],
    '@events'    => [
        'input' => 'onInput',
        'blur'  => 'validate'
    ]
]) ?>
```

Mirip Vue:
```
<Input
  v-model="email"
  type="email"
  required
  placeholder="Masukkan email"
  @input="onInput"
/>
```

## 🧠 Konsep Mapping Vue → CI3
| Vue            | CI3 Component  |
| -------------- | -------------- |
| props          | `$props` array |
| default props  | `$defaults`    |
| v-bind="attrs" | `:attrs`       |
| @event         | `@events`      |
| v-model        | `modelValue`   |
| slot           | `$slot`        |
| class binding  | `class`        |
| boolean props  | `true / false` |


## 1️⃣ Upgrade Helper Component (Core Engine)

application/helpers/component_helper.php
```
<?php
defined('BASEPATH') OR exit('No direct script access allowed');

function component($name, $props = [], $slot = null)
{
    $CI =& get_instance();

    $data = [
        'props' => $props,
        'slot'  => $slot
    ];

    return $CI->load->view("components/{$name}", $data, TRUE);
}
```

👉 Sekarang SEMUA props dikirim mentah (seperti Vue).


## 2️⃣ Utility Helper (Vue-like Binding)
Tambahkan helper kecil:
```
function attr_string($attrs = [])
{
    $html = '';
    foreach ($attrs as $key => $value) {
        if ($value === true) {
            $html .= " {$key}";
        } elseif ($value !== false && $value !== null) {
            $html .= " {$key}=\"".html_escape($value)."\"";
        }
    }
    return $html;
}
```


## 3️⃣ Component Input (FULL Vue Props Support)
views/components/input.php
```
<?php
// === DEFAULT PROPS (Vue style) ===
$defaults = [
    'type'       => 'text',
    'name'       => null,
    'label'      => null,
    'modelValue' => '',
    'class'      => '',
    'required'   => false,
    'disabled'   => false,
];

// merge props
$p = array_merge($defaults, $props);

// extra bindings
$attrs  = $props[':attrs'] ?? [];
$events = $props['@events'] ?? [];
$error  = $props['error'] ?? null;

// generate event attrs
$eventAttrs = [];
foreach ($events as $event => $handler) {
    $eventAttrs["on{$event}"] = "{$handler}(this)";
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
        <?= attr_string([
            'required' => $p['required'],
            'disabled' => $p['disabled'],
            ...$attrs,
            ...$eventAttrs
        ]) ?>
        class="form-control <?= $error ? 'is-invalid' : '' ?>"
    >

    <?php if ($error): ?>
        <small style="color:red"><?= $error ?></small>
    <?php endif; ?>
</div>
```

🔥 Ini sudah mendekati engine Vue props


## 4️⃣ Component Card (Slot 100%)
```
<div class="card">
    <?php if (!empty($props['title'])): ?>
        <h3><?= html_escape($props['title']) ?></h3>
    <?php endif; ?>

    <?= $slot ?>
</div>
```


## 5️⃣ JavaScript (Event & Model Simulation)
assets/js/components.js
```
function onInput(el) {
  console.log('value:', el.value);
}

function validate(el) {
  if (!el.value) {
    el.style.borderColor = 'red';
  }
}
```

6️⃣ Boolean Props (Vue Compatible)

Vue:
```
<Input disabled />
```

CI3:
```
<?= component('input', [
    'disabled' => true
]) ?>
```

✔ otomatis jadi:
```
<input disabled>
```


## 7️⃣ Dynamic Attribute Binding (v-bind)

Vue:
```
<Input v-bind="attrs" />
```


CI3:
```
<?= component('input', [
    ':attrs' => [
        'placeholder' => 'Email',
        'maxlength' => 50,
        'data-id' => 12
    ]
]) ?>
```


## 8️⃣ Event Binding (@click, @input)

Vue:
```
<Input @input="onInput" />
```

CI3:
```
<?= component('input', [
    '@events' => [
        'input' => 'onInput'
    ]
]) ?>
```

## 9️⃣ Supported Vue Props (Checklist)
| Vue Feature     | Status        |
| --------------- | ------------- |
| props           | ✅             |
| default props   | ✅             |
| boolean props   | ✅             |
| slot            | ✅             |
| v-bind attrs    | ✅             |
| @events         | ✅             |
| class binding   | ✅             |
| v-model (basic) | ✅             |
| emits           | ⚠️ JS only    |
| reactivity      | ❌ (JS needed) |
| computed        | ❌             |


## 🧠 Kesimpulan Jujur

✔ Ini bukan Vue, tapi:
* Pola sama
* Cara pakai sama
* Struktur sama
* Developer experience mirip

🔥 Sangat cocok untuk:
* CI3 legacy
* Tanpa build tools
* Team PHP murni
* SSR + progressive enhancement
