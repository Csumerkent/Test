# %%{init: {'theme':'base'}}%%
flowchart TD

%% =========================
%% SYSTEM GROUPS
%% =========================

subgraph CRM["Dynamics CRM (Gösterim / Master Veri)"]
  CRM_PROFILES["CRM Profilleri<br/>CRMID + GlobalID"]
  CRM_MERGE{"Merge gerekli mi?"}
  CRM_MERGE_DECIDE["CRM'de Merge Kararı<br/>Master / Duplicate"]
  CRM_VIEW["CRM'e Yaz<br/>Puan + Üyelik"]
end

subgraph OPERA["Opera Cloud (PMS)"]
  OPERA_IMPORT["Profil Import<br/>(CRM Excel → Opera)<br/>Resort Manager"]
  OPERA_NAMEIDS["Opera NAMEID Al"]
  OPERA_EVENTS["Profil Event / Sync"]
  OPERA_UPDATE_CRMID["Opera'ya CRMID Yaz"]
  OPERA_APPLY_MERGE["Opera'da Merge Uygula"]
  OPERA_POINTS["Opera'ya Puan / Üyelik Yaz"]
end

subgraph S8["Suite8 (PMS)"]
  S8_APPLY_MERGE["Suite8'de Merge Uygula"]
  S8_POINTS["Suite8'e Puan / Üyelik Yaz"]
end

subgraph SCHED["MPY Loyalty Scheduler"]
  SCH_START["Günlük Scheduler"]
  SCH_CHECKOUT{"Checkout olacak<br/>profil var mı?"}
  SCH_FETCH["Checkout Profilleri Çek<br/>(Opera + Suite8)"]
  SCH_HISTORY["Diğer Rezervasyonları Bul"]
  SCH_RULES["Üyelik Kuralı Seç<br/>Voyage / Maxx"]
  SCH_CALC["Puan Hesapla<br/>Üyelik Güncelle"]
end

%% =========================
%% MAIN FLOW (ORDERED)
%% =========================

CRM_PROFILES --> OPERA_IMPORT
OPERA_IMPORT --> OPERA_NAMEIDS

OPERA_NAMEIDS --> MAP["Bizim Eşleştirme<br/>NAMEID ↔ CRMID"]
MAP --> OPERA_EVENTS

OPERA_EVENTS --> CRM_VIEW
CRM_VIEW --> OPERA_UPDATE_CRMID

%% Two-way sync
OPERA_EVENTS <--> CRM_VIEW
CRM_VIEW <--> OPERA_UPDATE_CRMID

%% Merge flow
CRM_VIEW --> CRM_MERGE
CRM_MERGE -- Hayır --> SCH_START
CRM_MERGE -- Evet --> CRM_MERGE_DECIDE
CRM_MERGE_DECIDE --> OPERA_APPLY_MERGE
CRM_MERGE_DECIDE --> S8_APPLY_MERGE

%% Scheduler flow
SCH_START --> SCH_CHECKOUT
SCH_CHECKOUT -- Hayır --> WAIT["Bekle / Ertesi Gün"]
SCH_CHECKOUT -- Evet --> SCH_FETCH
SCH_FETCH --> SCH_HISTORY
SCH_HISTORY --> SCH_RULES
SCH_RULES --> SCH_CALC

%% Result distribution
SCH_CALC --> CRM_VIEW
SCH_CALC --> OPERA_POINTS
SCH_CALC --> S8_POINTS

%% =========================
%% STYLES (FINAL LOOK)
%% =========================

style CRM fill:#E3F2FD,stroke:#1E88E5,stroke-width:2.5px,font-size:14px
style OPERA fill:#E6F4EA,stroke:#2E7D32,stroke-width:2.5px,font-size:14px
style S8 fill:#E6F4EA,stroke:#2E7D32,stroke-width:2.5px,font-size:14px
style SCHED fill:#F4ECF7,stroke:#8E44AD,stroke-width:2.5px,font-size:14px

style CRM_MERGE fill:#FFF1E6,stroke:#EF6C00,stroke-width:3px,font-weight:600
style SCH_CHECKOUT fill:#FFF1E6,stroke:#EF6C00,stroke-width:3px,font-weight:600

linkStyle default stroke:#546E7A,stroke-width:2px
