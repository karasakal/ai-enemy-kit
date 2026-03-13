# FPS Kit × AI Enemy Kit — Integration Guide

Bu rehber, **FPS Kit** ve **AI Enemy Kit**'i aynı Unity projesinde kullanmak için gereken tüm adımları kapsar.

---

## ✅ Önkoşullar

- FPS Kit `Assets/CarKit/` klasöründe import edilmiş
- AI Enemy Kit `Assets/AIEnemyKit/` klasöründe import edilmiş
- Her iki paketin script'leri aynı projede derleniyor

---

## 1. IDamageable — Otomatik Silah Entegrasyonu

FPS Kit'in `WeaponController.cs` zaten `IDamageable` interface'ini kullanır.
`EnemyHealth.cs` bu interface'i implement ettiği için **sıfır ek kod** ile çalışır.

```csharp
// EnemyHealth.cs (AI Enemy Kit) — zaten implement edilmiş:
public class EnemyHealth : MonoBehaviour, IDamageable
{
    public void TakeDamage(float damage, Vector3 hitPoint,
                           Vector3 hitNormal, string hitTag)
    {
        // hitTag: "Head" → headMultiplier, "Limb" → limbMultiplier, default → bodyMultiplier
        ...
    }
}
```

**Yapılacak tek şey:** Düşman objelerine `Collider` ekle ve tag'lerini ayarla:

| Collider | Tag |
|----------|-----|
| Kafa | `Head` |
| Gövde | `Body` *(veya boş bırak)* |
| Kol / Bacak | `Limb` |

FPS Kit raycast'i çarptığında `IDamageable.TakeDamage()` otomatik çağrılır.

---

## 2. Ses Algılama — Silah Sesi → Düşman İşitmesi

FPS Kit her atışta `WeaponController.OnFired` event'i fırlatır.
Bu event'i `SoundEmitter.Emit()` ile bağla:

### Adım 1 — SoundEmitter Ekle

```
WeaponController GameObject
└── SoundEmitter   ← bu komponenti ekle
```

Inspector'da:
- `Emit Duration`: `0.1`
- `Is Suppressed`: *(silaha göre)* — WeaponController'dan runtime'da set edilebilir

### Adım 2 — Event Bağlantısı

```csharp
// Bir bootstrap script veya WeaponManager'a ekle:
public class FPSKitAIBridge : MonoBehaviour
{
    private WeaponController _weapon;
    private SoundEmitter     _emitter;

    private void Awake()
    {
        _weapon  = GetComponent<WeaponController>();
        _emitter = GetComponent<SoundEmitter>();

        _weapon.OnFired += OnWeaponFired;
    }

    private void OnWeaponFired()
    {
        if (_emitter == null) return;
        // Silah susturucusu varsa suppressed yap
        // _emitter.IsSuppressed = _weapon.hasSuppressor;
        _emitter.Emit();
    }

    private void OnDestroy()
    {
        if (_weapon) _weapon.OnFired -= OnWeaponFired;
    }
}
```

Düşmanların `EnemyPerception` bileşeni `SoundEmitter`'ı otomatik algılar.
Ek event bağlantısı gerekmez.

---

## 3. Kamera Sarsıntısı — Grenade Patlaması

AI Enemy Kit grenade fırlattığında `OnGrenadeThrownAt` event'i fırlatılır.
FPS Kit kamerasına sarsıntı eklemek için:

```csharp
// Grenade patlama script'ine ekle:
public class GrenadeExplosion : MonoBehaviour
{
    public float shakeRadius    = 15f;
    public float shakeIntensity = 0.6f;

    private void Explode()
    {
        // ... patlama hasarı ...

        // FPS Kit kamerası
        var cam = FindObjectOfType<FPSCameraController>();
        if (cam != null)
        {
            float dist = Vector3.Distance(transform.position,
                         cam.transform.position);
            float falloff = Mathf.Clamp01(1f - dist / shakeRadius);
            cam.AddTrauma(shakeIntensity * falloff);
        }
    }
}
```

---

## 4. EnemyManager ↔ Oyuncu Referansı

EnemyManager'ın oyuncuyu takip etmesi için `playerTransform` alanını doldur.

**Inspector:** `EnemyManager → Player Transform` → FPS Kit player objeni sürükle

**Runtime (dinamik):**
```csharp
EnemyManager.Instance.playerTransform = player.transform;
```

---

## 5. Squad Alert — Oyuncu Tespit → Tüm Grup Uyarısı

FPS Kit oyuncusu görüldüğünde EnemyManager tüm gruptaki düşmanları uyarabilir:

```csharp
// Örnek: oyuncu tespit edilince
enemyAI.OnTargetAcquired += (target) =>
{
    // Global alarm — tüm düşmanlar uyarılır
    EnemyManager.Instance.AlertAllEnemies(target, target.position);
};
```

---

## 6. Hasar UI — FPS Kit HUD'una Düşman Durumu

FPS Kit'in `DemoHUD.cs`'ini genişleterek düşman sayısını göster:

```csharp
// DemoHUD.cs içine ekle:
private void Update()
{
    // ... mevcut FPS HUD kodu ...

    if (EnemyManager.Instance != null)
    {
        int alive = EnemyManager.Instance.ActiveEnemyCount;
        int killed = EnemyManager.Instance.TotalKilled;
        enemyCountText.text = $"Enemies: {alive}  Kills: {killed}";
    }
}
```

---

## 7. Hierarchy Örneği (İkisi Birlikte)

```
Scene
├── === FPS KIT ===
│   ├── FPSPlayer
│   │   ├── CharacterController_FPS
│   │   ├── FPSCameraController
│   │   ├── WeaponManager
│   │   │   └── Weapon_Rifle
│   │   │       ├── WeaponController
│   │   │       ├── SoundEmitter         ← Yeni eklendi
│   │   │       └── FPSKitAIBridge       ← Yeni eklendi
│   │   └── HUD_Canvas
│   │       └── DemoHUD
│
├── === AI ENEMY KIT ===
│   ├── EnemyManager
│   │   └── (playerTransform → FPSPlayer)
│   │
│   ├── Enemy_Infantry_01
│   │   ├── EnemyAI
│   │   ├── EnemyPerception
│   │   ├── EnemyHealth               ← IDamageable — FPS Kit uyumlu
│   │   ├── EnemyWeapon
│   │   ├── NavMeshAgent
│   │   └── Colliders
│   │       ├── HeadCollider  (tag: Head)
│   │       ├── BodyCollider  (tag: Body)
│   │       └── LimbCollider  (tag: Limb)
│   │
│   └── CoverPoints / SpawnPoints / PatrolRoutes
```

---

## 8. Hızlı Kontrol Listesi

- [ ] `Assets/CarKit/` ve `Assets/AIEnemyKit/` aynı projede
- [ ] Düşman collider'larına `Head` / `Body` / `Limb` tag'leri eklendi
- [ ] `SoundEmitter` WeaponController objesine eklendi
- [ ] `FPSKitAIBridge` WeaponController objesine eklendi
- [ ] `EnemyManager.playerTransform` FPS Kit oyuncusuna atandı
- [ ] NavMesh bake edildi (`Window → AI → Navigation → Bake`)
- [ ] `CoverPoint` objeleri sahneye yerleştirildi

---

## 9. Sık Karşılaşılan Sorunlar

**Düşman hasarı almıyor:**
→ Düşman collider'ının `IDamageable` içeren objeye veya parent'ına ulaşıldığını kontrol et.
→ FPS Kit `hitMask`'inin düşman layer'ını içerdiğinden emin ol.

**Düşman silah sesini duymuyor:**
→ `SoundEmitter.Emit()` her atışta çağrılıyor mu kontrol et.
→ `EnemyPerception.hearingRadius` değerini artır.

**NavMeshAgent yolu bulamıyor:**
→ NavMesh bake etmeyi unutma.
→ Düşman spawn pozisyonunun NavMesh üzerinde olduğundan emin ol.

**Kamera sarsıntısı çalışmıyor:**
→ `FPSCameraController` referansını runtime'da bul: `FindObjectOfType<FPSCameraController>()`

---

*Haner Games — hanergames.com.tr*
*FPS Kit × AI Enemy Kit Integration Guide v1.0*
