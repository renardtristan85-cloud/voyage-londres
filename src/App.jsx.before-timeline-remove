import { useState, useEffect, useCallback, useRef } from "react";
import { Plus, MapPin, X, ChevronDown, ChevronUp, Landmark, Lock, FileText, Trash2 } from "lucide-react";
import { supabase } from "./lib/supabase";


// ---------- Couleurs ----------
const C = {
  ink: "#101419",
  inkPanel: "#171C21",
  bone: "#F3F1EA",
  boneAlt: "#E8E6DF",
  signal: "#A71930",
  brass: "#A89468",
  graphite: "#4E555D",
  line: "rgba(16,20,25,0.14)",
  lineOnInk: "rgba(243,241,234,0.18)",
};
const serif = { fontFamily: "'Fraunces', Georgia, serif" };
const mono = { fontFamily: "'IBM Plex Mono', monospace" };

// ---------- Données de vote ----------
const DB_CATEGORY_BY_UI = {
  hebergement: "lodging",
  repas: "meal",
  activites: "activity",
};
const LODGING = [
  { id: "clink-dorm", name: "Clink261 — dortoir", price: "17–20 €", where: "261 Gray's Inn Road, WC1X 8QT", desc: "Auberge installée dans un ancien tribunal victorien, à 5 minutes de St Pancras. Lits en dortoir partagé — le tarif le plus bas du dossier." },
  { id: "clink-room4", name: "Clink261 — chambre privée (4)", price: "25–35 €", where: "261 Gray's Inn Road, WC1X 8QT", desc: "Même établissement, en chambre fermée pour quatre. Le groupe reste entre lui, plus simple à encadrer le soir." },
  { id: "stchris", name: "St Christopher's Inn — dortoir", price: "20–30 €", where: "165 Borough High St, SE1 1HP", desc: "Auberge rive sud, à deux pas de Tower Bridge et de HMS Belfast." },
  { id: "crestfield", name: "Crestfield Hotel", price: "≈ 70 €", where: "Près de King's Cross", desc: "Hôtel budget classique, petit-déjeuner inclus, proche de la gare d'arrivée." },
  { id: "central", name: "Central Hotel London", price: "70–80 €", where: "Près de King's Cross", desc: "B&B familial, petit-déjeuner inclus, ambiance plus feutrée qu'une auberge." },
  { id: "imperial", name: "Imperial Hotels (Bloomsbury)", price: "70–85 €", where: "Bloomsbury", desc: "Habitué des groupes scolaires, à côté du British Museum, chambres quadruples." },
];

const MEALS = [
  { id: "tesco", name: "Tesco meal deal", price: "≈ 4,50 €", where: "N'importe quel Tesco Express", desc: "Sandwich ou plat + snack + boisson — la solution la plus simple en autonomie." },
  { id: "restaurant", name: "Restaurant (Wahaca, Pizza Pilgrims…)", price: "15–18 €", where: "Selon le soir", desc: "Repas assis, plus convivial mais plus cher que la formule Tesco." },
  { id: "mix", name: "Formule mixte : Tesco + 1 restaurant / semaine", price: "46–49 € / semaine", where: "—", desc: "Le compromis actuellement retenu dans le budget du dossier : les repas en Tesco meal deal la plupart du temps, et une vraie sortie restaurant dans la semaine." },
];

const ACTIVITIES_CULTURE = [
  { id: "britishmuseum", name: "British Museum", price: "Gratuit", where: "Great Russell St, WC1B 3DG", desc: "L'un des plus grands musées du monde, collections couvrant toutes les civilisations." },
  { id: "westminster", name: "Westminster (Big Ben, Parlement, Abbaye)", price: "Extérieurs gratuits, abbaye ~29 £", where: "Parliament Square", desc: "Passage devant les grands monuments institutionnels britanniques ; l'abbaye se visite en option payante." },
  { id: "buckingham", name: "Buckingham Palace", price: "Gratuit (extérieur)", where: "The Mall", desc: "Passage devant la résidence royale, éventuellement au moment de la relève de la garde selon l'horaire." },
  { id: "nationalgallery", name: "National Gallery", price: "Gratuit", where: "Trafalgar Square, WC2N 5DN", desc: "Peinture européenne du XIIIe au XIXe siècle — option pour la matinée libre du dernier jour." },
  { id: "camden", name: "Camden Market", price: "Gratuit hors achats", where: "Camden Town", desc: "Marché couvert, street food et ambiance alternative." },
  { id: "towerbridge", name: "Tower Bridge", price: "Gratuit (extérieur), visite intérieure en option", where: "Tower Bridge Rd", desc: "Vue sur le pont depuis l'extérieur ; la visite intérieure (passerelles vitrées, salle des machines) est payante en option." },
];

const ACTIVITIES_SECURITY = [
  {
    id: "oldbailey",
    name: "Old Bailey (Central Criminal Court)",
    price: "Gratuit",
    where: "Old Bailey, London EC4M 7EH",
    desc: "Tribunal pénal central de Londres : galerie publique gratuite pour observer de vrais procès en direct — le cœur du volet sécurité du séjour.",
    meta: [
      { label: "Horaires", value: "Lun-ven, 10h-12h40 et 14h-15h40" },
      { label: "Conditions", value: "14 ans minimum, pièce d'identité obligatoire, aucun appareil électronique" },
      { label: "Contact", value: "info@oldbaileyinsight.co.uk (pack pédagogique groupe)" },
    ],
  },
  {
    id: "hmsbelfast",
    name: "HMS Belfast",
    price: "20–24 £ / pers",
    where: "The Queen's Walk, London SE1 2JH",
    desc: "Ancien croiseur de la Royal Navy amarré sur la Tamise : visite du navire et de ses équipements militaires, pont par pont.",
    meta: [{ label: "Contact", value: "iwm.org.uk (rubrique groupes scolaires)" }],
  },
  {
    id: "iwm",
    name: "Imperial War Museum",
    price: "Gratuit",
    where: "Lambeth Road, London SE1 6HZ",
    desc: "Musée dédié à l'histoire militaire britannique : armée, renseignement, conflits du XXe-XXIe siècle.",
    meta: [{ label: "Contact", value: "iwm.org.uk" }],
  },
  {
    id: "firestation",
    name: "Caserne de pompiers (visite à confirmer)",
    price: "Gratuit",
    where: "Euston (172 Euston Rd) / Soho (126 Shaftesbury Ave) / Dowgate (94-95 Upper Thames St)",
    desc: "Découverte d'une caserne active : matériel et organisation des secours.",
    meta: [
      { label: "Contact", value: "london-fire.gov.uk — « Book your school visit »" },
      { label: "À savoir", value: "Demande à faire plusieurs mois à l'avance, non garantie le jour J" },
    ],
  },
];

const TABS = [
  { key: "hebergement", label: "Hébergement", chapter: "01", subtitle: "STAY IN LONDON" },
  { key: "repas", label: "Repas", chapter: "02", subtitle: "TASTE OF LONDON" },
  { key: "activites", label: "Activités", chapter: "03", subtitle: "DISCOVER LONDON" },
  { key: "dossier", label: "Dossier", chapter: "04", subtitle: "TRAVEL FILE" },
];

const VOTE_CATEGORIES = ["hebergement", "repas", "activites"];

const emptyTripData = () => ({
  votes: { hebergement: {}, repas: {}, activites: {} },
  customActivities: [],
  customMeals: [],
  logs: [],
});

// Convertit un ancien format de votes (liste de noms = "oui" implicite)
// vers le nouveau format { nom: "oui" | "non" }, pour ne pas perdre les
// choix déjà faits avec la version précédente du site.
function normalizeData(raw) {
  const base = { ...emptyTripData(), ...(raw || {}) };
  const votes = {};
  VOTE_CATEGORIES.forEach((cat) => {
    const catVotes = base.votes?.[cat] || {};
    const norm = {};
    Object.entries(catVotes).forEach(([itemId, val]) => {
      if (Array.isArray(val)) {
        norm[itemId] = {};
        val.forEach((n) => (norm[itemId][n] = "oui"));
      } else if (val && typeof val === "object") {
        norm[itemId] = val;
      }
    });
    votes[cat] = norm;
  });
  return { ...base, votes };
}

// ---------- Petits composants ----------

  const voyageTimeline = [
    { date: "13 MAR", title: "PARIS → LONDON", detail: "Départ · Arrivée à Londres" },
    { date: "14 MAR", title: "LONDON", detail: "Première journée · Découverte" },
    { date: "15 MAR", title: "LONDON", detail: "Culture · Activités · Centre-ville" },
    { date: "16 MAR", title: "LONDON", detail: "Dernière journée · Programme libre" },
    { date: "17 MAR", title: "LONDON → PARIS", detail: "Retour · Fin du voyage" },
  ];

function Barcode({ height = 22 }) {
  return (
    <div
      style={{
        height,
        backgroundImage:
          "repeating-linear-gradient(90deg, currentColor 0px, currentColor 2px, transparent 2px, transparent 5px, currentColor 5px, currentColor 6px, transparent 6px, transparent 10px)",
        opacity: 0.85,
      }}
    />
  );
}

function RoutePath() {
  return (
    <svg width="230" height="64" viewBox="0 0 230 64" fill="none">
      <line x1="14" y1="32" x2="216" y2="32" stroke={C.lineOnInk} strokeWidth="1.5" strokeDasharray="1 7" strokeLinecap="round" />
      <circle cx="14" cy="32" r="5" fill={C.bone} />
      <circle cx="216" cy="32" r="5" fill={C.signal} />
      <text x="0" y="16" fill={C.bone} style={{ ...mono, fontSize: 11, letterSpacing: 1 }}>PARIS</text>
      <text x="178" y="16" fill={C.signal} style={{ ...mono, fontSize: 11, letterSpacing: 1 }}>LONDRES</text>
      <text x="0" y="52" fill={C.graphite} style={{ ...mono, fontSize: 10 }}>CDG</text>
      <text x="196" y="52" fill={C.graphite} style={{ ...mono, fontSize: 10 }}>STP</text>
    </svg>
  );
}

function VoteStamp({ label, active, disabled, onClick, kind }) {
  const isOui = kind === "oui";

  return (
    <button
      type="button"
      onClick={onClick}
      disabled={disabled}
      className={`vote-button ${isOui ? "vote-oui" : "vote-non"} ${active ? "active" : ""}`}
      style={{ cursor: disabled ? "not-allowed" : "pointer" }}
    >
      <span className="vote-icon">{isOui ? "✓" : "×"}</span>
      <span className="vote-label">{label}</span>
    </button>
  );
}

function Ticket({ item, category, itemVotes, voterName, onVote, isAdmin, onDelete }) {
  const [open, setOpen] = useState(false);
  const [confirmDelete, setConfirmDelete] = useState(false);
  const votesObj = itemVotes || {};
  const ouiNames = Object.entries(votesObj).filter(([, v]) => v === "oui").map(([n]) => n);
  const nonNames = Object.entries(votesObj).filter(([, v]) => v === "non").map(([n]) => n);
  const myVote = voterName ? votesObj[voterName] : undefined;
  const long = item.desc.length > 130;
  const canDelete = item.author && onDelete && (isAdmin || (voterName && voterName === item.author));

  return (
    <div className={`ticket-card ticket-category-${category}`} style={{ background: C.bone, border: `1px solid ${C.line}`, boxShadow: `6px 6px 0 rgba(11,15,20,0.14)` }}>
      <div className="ticket-watermark" aria-hidden="true" />
      <div className="flex-1 p-6 flex flex-col gap-3">
        <div className="ticket-header-premium">
          <div className="ticket-header-top">
            <span>VOYAGE LONDRES · PASSENGER DOSSIER</span>
            <span>EUROSTAR 9020</span>
          </div>

          <div className="ticket-header-rule" />

          <div className="flex items-start justify-between gap-3">
            <div>
              <h3 style={{ ...serif, fontWeight: 600, fontSize: 21, color: C.ink }}>{item.name}</h3>
              <div className="flex items-center gap-1.5 mt-1.5" style={{ color: C.graphite, fontSize: 12, ...mono }}>
                <MapPin size={12} /> {item.where}
              </div>
            </div>
            {item.author && (
              <span className="shrink-0 px-2 py-1" style={{ border: `1px solid ${C.brass}`, color: C.brass, fontSize: 10.5, letterSpacing: 1, ...mono }}>
                PROPOSITION
              </span>
            )}
          </div>

          <div className="ticket-header-bottom">
            <span>13 MAR — 17 MAR 2028</span>
            <span>PARIS → LONDON</span>
          </div>
        </div>
        <p style={{ color: C.ink, fontSize: 14.5, lineHeight: 1.55, opacity: 0.85, maxWidth: "60ch" }}>
          {long && !open ? item.desc.slice(0, 128) + "…" : item.desc}
        </p>
        {long && (
          <button onClick={() => setOpen(!open)} className="flex items-center gap-1 self-start" style={{ color: C.brass, fontSize: 12, ...mono }}>
            {open ? <>Réduire le descriptif <ChevronUp size={13} /></> : <>Lire le descriptif <ChevronDown size={13} /></>}
          </button>
        )}
        <div className="ticket-route-premium">
          <div className="ticket-route-head">
            <span>PARIS</span>
            <span className="ticket-route-date">13—17 MAR 2028</span>
            <span>LONDON</span>
          </div>

          <div className="ticket-route-line">
            <span className="ticket-route-dot ticket-route-dot-start" />
            <span className="ticket-route-track" />
            <span className="ticket-route-dot ticket-route-dot-end" />
          </div>

          <div className="ticket-route-codes">
            <span>CDG</span>
            <span>STP</span>
          </div>

          <div className="ticket-microprint">
            PARIS × LONDON · EUROSTAR 9020 · PASSENGER DOSSIER · 2028
          </div>
        </div>

        {item.meta && (
          <div className="flex flex-col gap-1 py-1">
            {item.meta.map((m, i) => (
              <div key={i} style={{ fontSize: 12, lineHeight: 1.5 }}>
                <span style={{ ...mono, color: C.brass, fontWeight: 500 }}>{m.label} — </span>
                <span style={{ color: C.ink, opacity: 0.8 }}>{m.value}</span>
              </div>
            ))}
          </div>
        )}
        {item.author && (
          <div className="pt-2 flex items-center justify-between gap-3" style={{ borderTop: `1px dashed ${C.line}` }}>
            <span style={{ ...serif, fontStyle: "italic", fontSize: 13.5, color: C.ink, opacity: 0.75 }}>— proposée par {item.author}</span>
            {canDelete && !confirmDelete && (
              <button onClick={() => setConfirmDelete(true)} style={{ color: C.graphite, fontSize: 11.5, ...mono, cursor: "pointer" }}>
                Retirer
              </button>
            )}
            {canDelete && confirmDelete && (
              <span className="flex items-center gap-2">
                <span style={{ color: C.signal, fontSize: 11.5, ...mono }}>Supprimer ?</span>
                <button onClick={onDelete} style={{ color: C.signal, fontSize: 11.5, ...mono, fontWeight: 600, cursor: "pointer" }}>
                  Oui
                </button>
                <button onClick={() => setConfirmDelete(false)} style={{ color: C.graphite, fontSize: 11.5, ...mono, cursor: "pointer" }}>
                  Annuler
                </button>
              </span>
            )}
          </div>
        )}
      </div>

      <div className="ticket-divider-desktop">
        <div className="ticket-divider-line" />
        <div className="ticket-notch ticket-notch-top" />
        <div className="ticket-notch ticket-notch-bottom" />
      </div>

      <div className="ticket-divider-mobile" />

      <div className="flex flex-col items-center justify-center gap-3 px-6 py-5 md:w-56" style={{ background: C.boneAlt }}>
        <div className="text-center">
          <div style={{ ...serif, fontWeight: 600, fontSize: 18, color: C.ink }}>{item.price}</div>
          <div style={{ ...mono, fontSize: 10, color: C.graphite, letterSpacing: 1 }}>par personne</div>
        </div>
        <div className="flex gap-2">
          <VoteStamp label="Oui" kind="oui" active={myVote === "oui"} disabled={!voterName} onClick={() => onVote("oui")} />
          <VoteStamp label="Non" kind="non" active={myVote === "non"} disabled={!voterName} onClick={() => onVote("non")} />
        </div>
        <div className="text-center" style={{ ...mono, fontSize: 10.5, color: C.graphite, lineHeight: 1.5 }}>
          {ouiNames.length === 0 && nonNames.length === 0 ? (
            "Aucun vote pour l'instant"
          ) : (
            <>
              {ouiNames.length > 0 && <div style={{ color: C.ink }}>Oui ({ouiNames.length}) : {ouiNames.join(", ")}</div>}
              {nonNames.length > 0 && <div>Non ({nonNames.length}) : {nonNames.join(", ")}</div>}
            </>
          )}
        </div>
      </div>
    </div>
  );
}

function SectionTitle({ icon, children }) {
  return (
    <div className="flex items-center gap-2 mt-2 mb-1">
      {icon}
      <h2 style={{ ...serif, color: C.bone, fontSize: 22, fontWeight: 600 }}>{children}</h2>
    </div>
  );
}

function BlankTicket({ onOpen, canOpen, label, placeholderName }) {
  const [open, setOpen] = useState(false);
  const [form, setForm] = useState({ name: "", price: "", where: "", desc: "" });
  const [error, setError] = useState("");

  if (!open) {
    return (
      <button
        onClick={() => canOpen && setOpen(true)}
        disabled={!canOpen}
        className="flex flex-col items-center justify-center gap-2 p-8 w-full"
        style={{ background: C.bone, border: `2px dashed ${C.brass}`, minHeight: 160, color: canOpen ? C.ink : C.graphite, opacity: canOpen ? 1 : 0.6, cursor: canOpen ? "pointer" : "not-allowed" }}
      >
        <Plus size={24} color={canOpen ? C.signal : C.graphite} />
        <span style={{ ...serif, fontSize: 17, fontWeight: 600 }}>{label}</span>
        {!canOpen && <span style={{ fontSize: 11, ...mono }}>Indique ton prénom pour proposer</span>}
      </button>
    );
  }

  const handleSubmit = () => {
    if (!form.name.trim() || !form.desc.trim()) {
      setError("Renseigne au moins le nom et le descriptif avant d'envoyer.");
      return;
    }
    onOpen(form);
    setForm({ name: "", price: "", where: "", desc: "" });
    setError("");
    setOpen(false);
  };

  return (
    <div className="p-6 flex flex-col gap-2.5" style={{ background: C.bone, border: `1px solid ${C.line}`, boxShadow: `6px 6px 0 rgba(11,15,20,0.14)` }}>
      <div className="flex items-center justify-between">
        <span style={{ ...serif, fontWeight: 600, fontSize: 17, color: C.ink }}>{label}</span>
        <button type="button" onClick={() => setOpen(false)} style={{ cursor: "pointer" }}>
          <X size={16} color={C.graphite} />
        </button>
      </div>
      <input autoFocus placeholder={placeholderName + " *"} value={form.name} onChange={(e) => setForm({ ...form, name: e.target.value })} className="px-3 py-2 text-sm outline-none" style={{ border: `1px solid ${C.line}`, background: "white", color: C.ink }} />
      <div className="flex gap-2">
        <input placeholder="Lieu" value={form.where} onChange={(e) => setForm({ ...form, where: e.target.value })} className="px-3 py-2 text-sm outline-none flex-1" style={{ border: `1px solid ${C.line}`, background: "white", color: C.ink }} />
        <input placeholder="Prix estimé" value={form.price} onChange={(e) => setForm({ ...form, price: e.target.value })} className="px-3 py-2 text-sm outline-none w-32" style={{ border: `1px solid ${C.line}`, background: "white", color: C.ink }} />
      </div>
      <textarea placeholder="Descriptif : en quoi ça consiste, pourquoi la proposer *" value={form.desc} onChange={(e) => setForm({ ...form, desc: e.target.value })} rows={3} className="px-3 py-2 text-sm outline-none resize-none" style={{ border: `1px solid ${C.line}`, background: "white", color: C.ink }} />
      {error && <div style={{ color: C.signal, fontSize: 12.5, ...mono }}>{error}</div>}
      <button type="button" onClick={handleSubmit} className="py-2.5 font-medium mt-1" style={{ background: C.signal, color: C.bone, fontSize: 13, ...mono, cursor: "pointer" }}>
        Envoyer la proposition
      </button>
    </div>
  );
}

function NameGate({ onSubmit, onSkip }) {
  const [val, setVal] = useState("");
  return (
    <div className="fixed inset-0 flex items-center justify-center p-6 z-50" style={{ background: "rgba(11,15,20,0.88)" }}>
      <div className="w-full max-w-sm" style={{ background: C.bone, border: `1px solid ${C.line}`, boxShadow: `8px 8px 0 rgba(11,15,20,0.3)` }}>
        <div className="p-8">
          <div style={{ ...mono, color: C.brass, fontSize: 11, letterSpacing: 2 }}>CARTE PASSAGER</div>
          <h2 style={{ ...serif, fontSize: 24, fontWeight: 600, color: C.ink, marginTop: 6 }}>À qui le billet ?</h2>
          <p style={{ color: C.graphite, fontSize: 13.5, marginTop: 6, marginBottom: 20, lineHeight: 1.5 }}>
            Ton prénom permet de voter Oui/Non sur l'hébergement, les repas et les activités, et de voir qui a voté quoi.
          </p>
          <div className="flex flex-col gap-3">
            <input autoFocus value={val} onChange={(e) => setVal(e.target.value)} onKeyDown={(e) => e.key === "Enter" && val.trim() && onSubmit(val.trim())} placeholder="Ton prénom" className="w-full px-3 py-2.5 outline-none" style={{ border: `1px solid ${C.line}`, background: "white", color: C.ink, fontSize: 14 }} />
            <button type="button" onClick={() => val.trim() && onSubmit(val.trim())} className="w-full py-2.5 font-medium" style={{ background: C.signal, color: C.bone, fontSize: 13.5, letterSpacing: 0.5, ...mono, cursor: "pointer" }}>
              Embarquer
            </button>
            <button type="button" onClick={onSkip} className="w-full py-1 text-center" style={{ color: C.graphite, fontSize: 12, ...mono, cursor: "pointer" }}>
              Feuilleter sans s'identifier
            </button>
          </div>
        </div>
        <div style={{ borderTop: `2px dashed ${C.line}`, padding: "10px 32px", color: C.ink }}>
          <Barcode height={18} />
        </div>
      </div>
    </div>
  );
}

function AdminGate({ onSubmit, onClose }) {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  const submit = async () => {
    if (!email.trim() || !password) {
      setError("Renseigne l'adresse e-mail et le mot de passe.");
      return;
    }

    setError("");
    setLoading(true);

    try {
      const ok = await onSubmit(email.trim(), password);

      if (!ok) {
        setError("Identifiants incorrects ou compte non autorisé.");
      }
    } catch {
      setError("Impossible de se connecter. Réessaie.");
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="fixed inset-0 flex items-center justify-center p-6 z-50" style={{ background: "rgba(11,15,20,0.88)" }}>
      <div className="w-full max-w-sm" style={{ background: C.bone, border: `1px solid ${C.line}`, boxShadow: `8px 8px 0 rgba(11,15,20,0.3)` }}>
        <div className="p-8">
          <div className="flex items-center gap-2">
            <Lock size={16} color={C.brass} />
            <div style={{ ...mono, color: C.brass, fontSize: 11, letterSpacing: 2 }}>ESPACE ORGANISATEUR</div>
          </div>

          <h2 style={{ ...serif, fontSize: 22, fontWeight: 600, color: C.ink, marginTop: 6 }}>
            Connexion organisateur
          </h2>

          <div className="flex flex-col gap-3 mt-4">
            <input
              autoFocus
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              onKeyDown={(e) => e.key === "Enter" && submit()}
              placeholder="Adresse e-mail"
              className="w-full px-3 py-2.5 outline-none"
              style={{ border: `1px solid ${C.line}`, background: "white", color: C.ink, fontSize: 14 }}
            />

            <input
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              onKeyDown={(e) => e.key === "Enter" && submit()}
              placeholder="Mot de passe"
              className="w-full px-3 py-2.5 outline-none"
              style={{ border: `1px solid ${C.line}`, background: "white", color: C.ink, fontSize: 14 }}
            />

            {error && (
              <div style={{ color: C.signal, fontSize: 12.5, ...mono }}>
                {error}
              </div>
            )}

            <button
              type="button"
              onClick={submit}
              disabled={loading}
              className="w-full py-2.5 font-medium"
              style={{
                background: loading ? C.graphite : C.signal,
                color: C.bone,
                fontSize: 13.5,
                ...mono,
                cursor: loading ? "wait" : "pointer",
              }}
            >
              {loading ? "Connexion..." : "Entrer"}
            </button>

            <button
              type="button"
              onClick={onClose}
              disabled={loading}
              className="w-full py-1 text-center"
              style={{ color: C.graphite, fontSize: 12, ...mono, cursor: "pointer" }}
            >
              Annuler
            </button>
          </div>
        </div>
      </div>
    </div>
  );
}
function fmtTime(ts) {
  try {
    return new Date(ts).toLocaleString("fr-FR", { day: "2-digit", month: "2-digit", hour: "2-digit", minute: "2-digit" });
  } catch {
    return "—";
  }
}

const CAT_LABELS = { hebergement: "Hébergement", repas: "Repas", activites: "Activités" };
const LOG_LABELS = {
  join: (l) => `${l.name} s'est connecté·e`,
  vote: (l) => `${l.name} a ${l.action} « ${l.itemName || l.itemId} » (${CAT_LABELS[l.category] || l.category})`,
  propose: (l) => `${l.name} a proposé « ${l.itemName} » (${l.category})`,
  remove: (l) => `${l.name} a retiré « ${l.itemName} »`,
};

function AdminPanel({ data, allItems, onClose, supabaseClient }) {
  const [adminParticipants, setAdminParticipants] = useState([]);
  const [adminVotes, setAdminVotes] = useState([]);
  const [adminLogs, setAdminLogs] = useState([]);
  const [adminOptions, setAdminOptions] = useState([]);
  const [adminProposals, setAdminProposals] = useState([]);
  const [adminLoading, setAdminLoading] = useState(true);

  useEffect(() => {
    let cancelled = false;

    const loadAdminData = async () => {
      setAdminLoading(true);

      const [
        participantsResult,
        votesResult,
        logsResult,
        optionsResult,
        proposalsResult,
      ] = await Promise.all([
        supabaseClient
          .from("participants")
          .select("*")
          .order("created_at", { ascending: true }),

        supabaseClient
          .from("votes")
          .select("*")
          .order("updated_at", { ascending: false }),

        supabaseClient
          .from("activity_logs")
          .select("*")
          .order("created_at", { ascending: false }),

        supabaseClient
          .from("options")
          .select("*")
          .order("created_at", { ascending: true }),

        supabaseClient
          .from("proposals")
          .select("*")
          .order("created_at", { ascending: false }),
      ]);

      if (cancelled) return;

      if (participantsResult.error) {
        console.error(
          "Lecture participants organisateur:",
          participantsResult.error
        );
      }

      if (votesResult.error) {
        console.error(
          "Lecture votes organisateur:",
          votesResult.error
        );
      }

      if (logsResult.error) {
        console.error(
          "Lecture journal organisateur:",
          logsResult.error
        );
      }

      if (optionsResult.error) {
        console.error(
          "Lecture options organisateur:",
          optionsResult.error
        );
      }

      if (proposalsResult.error) {
        console.error(
          "Lecture propositions organisateur:",
          proposalsResult.error
        );
      }

      setAdminParticipants(participantsResult.data || []);
      setAdminVotes(votesResult.data || []);
      setAdminLogs(logsResult.data || []);
      setAdminOptions(optionsResult.data || []);
      setAdminProposals(proposalsResult.data || []);

      setAdminLoading(false);
    };

    loadAdminData();

    const interval = setInterval(loadAdminData, 5000);

    return () => {
      cancelled = true;
      clearInterval(interval);
    };
  }, [supabaseClient]);
  const participantsById = new Map(
    adminParticipants.map((participant) => [
      participant.id,
      participant,
    ])
  );

  const logs = adminLogs.map((log) => {
    const participant = participantsById.get(log.participant_id);
    const details = log.details || {};

    return {
      ...log,
      name: participant?.name || "Inconnu",
      type: log.action,
      at: log.created_at
        ? new Date(log.created_at).getTime()
        : null,
      itemName: details.item_name,
      choice: details.choice,
      category: details.category,
      result: details.result,
    };
  });

  const optionsById = new Map(
    adminOptions.map((option) => [
      option.id,
      option,
    ])
  );

  const passengers = adminParticipants.map((participant) => {
    const ownLogs = adminLogs.filter(
      (log) => log.participant_id === participant.id
    );

    const ownVotes = adminVotes.filter(
      (vote) => vote.participant_id === participant.id
    );

    const selections = ownVotes
      .filter((vote) => vote.choice === true)
      .map((vote) => {
        const option = optionsById.get(vote.option_id);
        return option ? option.title : vote.option_id;
      });

    const answered = {};

    VOTE_CATEGORIES.forEach((cat) => {
      const dbCategory = DB_CATEGORY_BY_UI[cat];

      answered[cat] = ownVotes.some((vote) => {
        const option = optionsById.get(vote.option_id);
        return option?.category === dbCategory;
      });
    });

    return {
      id: participant.id,
      name: participant.name,
      visits: ownLogs.filter(
        (log) => log.action === "join"
      ).length,
      lastSeen: participant.last_seen
        ? new Date(participant.last_seen).getTime()
        : null,
      selections,
      answered,
    };
  });
  const deleteOption = async (optionId) => {
    const option = adminOptions.find((item) => item.id === optionId);

    const confirmed = window.confirm(
      `Supprimer définitivement "${option?.title || "cette option"}" ?\\n\\nLes votes associés seront également supprimés.`
    );

    if (!confirmed) return;

    const { error } = await supabaseClient
      .from("options")
      .delete()
      .eq("id", optionId);

    if (error) {
      console.error("Suppression option:", error);
      window.alert("Impossible de supprimer cette option.");
      return;
    }

    setAdminOptions((current) =>
      current.filter((item) => item.id !== optionId)
    );

    setAdminVotes((current) =>
      current.filter((vote) => vote.option_id !== optionId)
    );
  };

  const deleteProposal = async (proposalId) => {
    const confirmed = window.confirm(
      "Supprimer définitivement cette proposition ?"
    );

    if (!confirmed) return;

    const { error } = await supabaseClient
      .from("proposals")
      .delete()
      .eq("id", proposalId);

    if (error) {
      console.error("Suppression proposition:", error);
      window.alert("Impossible de supprimer cette proposition.");
      return;
    }

    setAdminProposals((current) =>
      current.filter((proposal) => proposal.id !== proposalId)
    );
  };

  return (
    <div className="fixed inset-0 z-50 overflow-y-auto" style={{ background: C.ink }}>
      <div className="px-6 md:px-14 py-10 max-w-4xl mx-auto">
        <div className="flex items-center justify-between mb-6">
          <div>
            <div style={{ ...mono, color: C.brass, fontSize: 11, letterSpacing: 2 }}>ESPACE ORGANISATEUR</div>
            <h1 style={{ ...serif, color: C.bone, fontSize: 32, fontWeight: 600 }}>Registre de bord</h1>
          </div>
          <button onClick={onClose} className="px-3 py-2 flex items-center gap-1" style={{ border: `1px solid ${C.lineOnInk}`, color: C.bone, fontSize: 12.5, ...mono, cursor: "pointer" }}>
            <X size={14} /> Fermer
          </button>
        </div>

        <div
          style={{
            background: C.bone,
            border: `1px solid ${C.line}`,
            boxShadow: `6px 6px 0 rgba(11,15,20,0.14)`,
          }}
          className="p-5 mb-6"
        >
          <h2
            style={{
              ...serif,
              fontWeight: 600,
              fontSize: 17,
              color: C.ink,
              marginBottom: 12,
            }}
          >
            Récapitulatif du voyage
          </h2>

          <div className="flex flex-col gap-5">
            {VOTE_CATEGORIES.map((cat) => {
              const dbCategory = DB_CATEGORY_BY_UI[cat];

              const options = adminOptions
                .filter((option) => option.category === dbCategory)
                .map((option) => {
                  const optionVotes = adminVotes.filter(
                    (vote) => vote.option_id === option.id
                  );

                  const yesCount = optionVotes.filter(
                    (vote) => vote.choice === true
                  ).length;

                  const noCount = optionVotes.filter(
                    (vote) => vote.choice === false
                  ).length;

                  const itemId =
                    Object.keys(allItems).find(
                      (id) => allItems[id]?.name === option.title
                    ) || option.id;

                  return {
                    itemId,
                    item: allItems[itemId] || {
                      name: option.title,
                    },
                    yesCount,
                    noCount,
                    total: yesCount + noCount,
                  };
                })
                .sort((a, b) => b.yesCount - a.yesCount);

              const totalYes = options.reduce(
                (sum, option) => sum + option.yesCount,
                0
              );

              const totalNo = options.reduce(
                (sum, option) => sum + option.noCount,
                0
              );

              const totalVotes = totalYes + totalNo;

              const maxYes = options.length
                ? Math.max(...options.map((option) => option.yesCount))
                : 0;

              const leaders = options.filter(
                (option) => option.yesCount === maxYes && maxYes > 0
              );

              let status = "PAS ENCORE DÉCIDÉ";

              if (leaders.length === 1 && maxYes > totalYes / 2) {
                status = "MAJORITÉ";
              } else if (leaders.length > 1) {
                status = "ÉGALITÉ";
              } else if (maxYes > 0) {
                status = "CHOIX EN TÊTE";
              }

              return (
                <div key={cat}>
                  <div
                    className="flex items-center justify-between gap-3 flex-wrap"
                    style={{ marginBottom: 8 }}
                  >
                    <div
                      style={{
                        ...mono,
                        fontSize: 10.5,
                        color: C.brass,
                        letterSpacing: 1,
                      }}
                    >
                      {CAT_LABELS[cat].toUpperCase()}
                    </div>

                    <div
                      style={{
                        ...mono,
                        fontSize: 9.5,
                        color: C.graphite,
                        border: `1px solid ${C.line}`,
                        padding: "3px 7px",
                      }}
                    >
                      {status}
                    </div>
                  </div>

                  <div
                    className="flex gap-4 flex-wrap"
                    style={{
                      fontSize: 11,
                      color: C.graphite,
                      marginBottom: 9,
                    }}
                  >
                    <span>{totalVotes} vote{totalVotes > 1 ? "s" : ""}</span>
                    <span>{totalYes} OUI</span>
                    <span>{totalNo} NON</span>
                  </div>

                  {options.length === 0 ? (
                    <div
                      style={{
                        fontSize: 12.5,
                        color: C.graphite,
                      }}
                    >
                      Aucun vote pour l'instant.
                    </div>
                  ) : (
                    <div className="flex flex-col gap-1.5">
                      {options.map(
                        ({ itemId, item, yesCount, noCount, total }) => {
                          const isLeader =
                            maxYes > 0 && yesCount === maxYes;

                          return (
                            <div
                              key={itemId}
                              className="flex items-center justify-between gap-3 py-2"
                              style={{
                                borderBottom: `1px solid ${C.line}`,
                              }}
                            >
                              <div className="min-w-0">
                                <div
                                  style={{
                                    fontSize: 13,
                                    color: C.ink,
                                    fontWeight: isLeader ? 600 : 400,
                                  }}
                                >
                                  {item.name}
                                  {isLeader && (
                                    <span
                                      style={{
                                        ...mono,
                                        fontSize: 9,
                                        color: C.brass,
                                        marginLeft: 7,
                                      }}
                                    >
                                      EN TÊTE
                                    </span>
                                  )}
                                </div>

                                <div
                                  style={{
                                    ...mono,
                                    fontSize: 9.5,
                                    color: C.graphite,
                                    marginTop: 2,
                                  }}
                                >
                                  {total} vote{total > 1 ? "s" : ""}
                                </div>
                              </div>

                              <div
                                className="flex gap-2"
                                style={{
                                  ...mono,
                                  fontSize: 10.5,
                                  whiteSpace: "nowrap",
                                }}
                              >
                                <span>{yesCount} OUI</span>
                                <span style={{ color: C.graphite }}>
                                  {noCount} NON
                                </span>
                              </div>
                            </div>
                          );
                        }
                      )}
                    </div>
                  )}
                </div>
              );
            })}
          </div>
        </div>

        <div
          style={{
            background: C.bone,
            border: `1px solid ${C.line}`,
            boxShadow: `6px 6px 0 rgba(11,15,20,0.14)`,
          }}
          className="p-5 mb-6"
        >
          <h2
            style={{
              ...serif,
              fontWeight: 600,
              fontSize: 17,
              color: C.ink,
              marginBottom: 4,
            }}
          >
            Historique des votes
          </h2>

          <p
            style={{
              fontSize: 12.5,
              color: C.graphite,
              marginBottom: 12,
            }}
          >
            Chaque changement de vote est conservé.
          </p>

          {logs.filter((log) => log.type === "vote").length === 0 ? (
            <div style={{ color: C.graphite, fontSize: 13 }}>
              Aucun vote enregistré.
            </div>
          ) : (
            <div className="flex flex-col gap-1.5 max-h-96 overflow-y-auto">
              {logs
                .filter((log) => log.type === "vote")
                .map((log) => {
                  const isYes = log.choice === "oui";

                  return (
                    <div
                      key={log.id}
                      className="py-2"
                      style={{
                        borderBottom: `1px solid ${C.line}`,
                      }}
                    >
                      <div
                        className="flex items-center justify-between gap-3 flex-wrap"
                      >
                        <div
                          style={{
                            fontSize: 12.5,
                            color: C.ink,
                          }}
                        >
                          <strong>{log.name}</strong>
                          {" — "}
                          {log.itemName || "Option inconnue"}
                        </div>

                        <span
                          style={{
                            ...mono,
                            fontSize: 9.5,
                            padding: "3px 7px",
                            border: `1px solid ${
                              isYes ? C.brass : C.line
                            }`,
                            color: isYes ? C.brass : C.graphite,
                          }}
                        >
                          {isYes ? "OUI" : "NON"}
                        </span>
                      </div>

                      <div
                        style={{
                          ...mono,
                          fontSize: 9.5,
                          color: C.graphite,
                          marginTop: 3,
                        }}
                      >
                        {log.at ? fmtTime(log.at) : "Date inconnue"}
                        {log.result ? ` · ${log.result}` : ""}
                      </div>
                    </div>
                  );
                })}
            </div>
          )}
        </div>

        <div style={{ background: C.bone, border: `1px solid ${C.line}`, boxShadow: `6px 6px 0 rgba(11,15,20,0.14)` }} className="p-5 mb-6">
          <h2 style={{ ...serif, fontWeight: 600, fontSize: 17, color: C.ink, marginBottom: 4 }}>Avancement</h2>
          <p style={{ fontSize: 12.5, color: C.graphite, marginBottom: 12 }}>{passengers.length} personne(s) connectée(s) au total.</p>
          <div className="grid grid-cols-3 gap-3 mb-4">
            {VOTE_CATEGORIES.map((cat) => {
              const count = passengers.filter((p) => p.answered[cat]).length;
              return (
                <div key={cat} className="text-center py-3" style={{ border: `1px solid ${C.line}` }}>
                  <div style={{ ...serif, fontSize: 22, fontWeight: 600, color: C.ink }}>{count}/{passengers.length || 0}</div>
                  <div style={{ ...mono, fontSize: 10.5, color: C.graphite }}>{CAT_LABELS[cat]}</div>
                </div>
              );
            })}
          </div>
          {passengers.length === 0 ? (
            <div style={{ color: C.graphite, fontSize: 13 }}>Personne ne s'est encore connecté.</div>
          ) : (
            <div className="flex flex-col divide-y" style={{ borderColor: C.line }}>
              {passengers
                .sort((a, b) => (b.lastSeen || 0) - (a.lastSeen || 0))
                .map((p) => (
                  <div key={p.name} className="py-3 flex flex-col gap-1.5">
                    <div className="flex items-center justify-between flex-wrap gap-2">
                      <span style={{ ...serif, fontWeight: 600, fontSize: 15, color: C.ink }}>{p.name}</span>
                      <span style={{ ...mono, fontSize: 11, color: C.graphite }}>
                        {p.visits} connexion{p.visits > 1 ? "s" : ""} · dernière : {p.lastSeen ? fmtTime(p.lastSeen) : "—"}
                      </span>
                    </div>
                    <div className="flex gap-2">
                      {VOTE_CATEGORIES.map((cat) => (
                        <span key={cat} style={{ ...mono, fontSize: 10, padding: "2px 7px", border: `1px solid ${p.answered[cat] ? C.brass : C.line}`, color: p.answered[cat] ? C.brass : C.graphite }}>
                          {p.answered[cat] ? "✓" : "—"} {CAT_LABELS[cat]}
                        </span>
                      ))}
                    </div>
                    <div style={{ fontSize: 12.5, color: C.ink, opacity: 0.8 }}>
                      {p.selections.length > 0 ? "Oui : " + p.selections.join(" · ") : "aucun oui pour l'instant"}
                    </div>
                  </div>
                ))}
            </div>
          )}
        </div>

        <div
          style={{
            background: C.bone,
            border: `1px solid ${C.line}`,
            boxShadow: `6px 6px 0 rgba(11,15,20,0.14)`,
          }}
          className="p-5 mb-6"
        >
          <div className="flex items-center justify-between gap-3 flex-wrap">
            <h2
              style={{
                ...serif,
                fontWeight: 600,
                fontSize: 17,
                color: C.ink,
                marginBottom: 4,
              }}
            >
              Tickets / options
            </h2>

            <span
              style={{
                ...mono,
                fontSize: 9.5,
                color: C.graphite,
                border: `1px solid ${C.line}`,
                padding: "3px 7px",
              }}
            >
              {adminOptions.length} option
              {adminOptions.length > 1 ? "s" : ""}
            </span>
          </div>

          <p
            style={{
              fontSize: 12.5,
              color: C.graphite,
              marginBottom: 14,
            }}
          >
            Toutes les options actuellement disponibles dans le voyage.
          </p>

          {adminOptions.length === 0 ? (
            <div style={{ color: C.graphite, fontSize: 13 }}>
              Aucune option enregistrée.
            </div>
          ) : (
            <div className="flex flex-col gap-2">
              {adminOptions.map((option) => (
                <div
                  key={option.id}
                  className="p-3"
                  style={{
                    border: `1px solid ${C.line}`,
                    background: "#F5F3EE",
                  }}
                >
                  <div className="flex items-start justify-between gap-4">
                    <div className="min-w-0">
                      <div
                        style={{
                          ...mono,
                          fontSize: 9,
                          color: C.brass,
                          letterSpacing: 1,
                          marginBottom: 3,
                        }}
                      >
                        {option.category?.toUpperCase() || "OPTION"}
                        {option.is_custom ? " · PROPOSITION" : ""}
                      </div>

                      <div
                        style={{
                          ...serif,
                          fontSize: 15,
                          fontWeight: 600,
                          color: C.ink,
                        }}
                      >
                        {option.title}
                      </div>

                      {option.description && (
                        <div
                          style={{
                            fontSize: 12,
                            color: C.graphite,
                            marginTop: 4,
                            lineHeight: 1.45,
                          }}
                        >
                          {option.description}
                        </div>
                      )}

                      <div
                        className="flex flex-wrap gap-3"
                        style={{
                          ...mono,
                          fontSize: 9.5,
                          color: C.graphite,
                          marginTop: 7,
                        }}
                      >
                        {option.location && (
                          <span>📍 {option.location}</span>
                        )}

                        {option.price !== null &&
                          option.price !== undefined && (
                            <span>{option.price} €</span>
                          )}

                        <span>
                          {option.created_at
                            ? fmtTime(
                                new Date(option.created_at).getTime()
                              )
                            : "Date inconnue"}
                        </span>
                      </div>
                    </div>

                    <button
                      type="button"
                      onClick={() => deleteOption(option.id)}
                      className="shrink-0 px-3 py-2"
                      style={{
                        border: `1px solid ${C.line}`,
                        color: C.ink,
                        background: "transparent",
                        fontSize: 11,
                        ...mono,
                        cursor: "pointer",
                      }}
                    >
                      <Trash2 size={13} className="inline mr-1" />
                      Supprimer
                    </button>
                  </div>
                </div>
              ))}
            </div>
          )}
        </div>

        <div
          style={{
            background: C.bone,
            border: `1px solid ${C.line}`,
            boxShadow: `6px 6px 0 rgba(11,15,20,0.14)`,
          }}
          className="p-5 mb-6"
        >
          <div className="flex items-center justify-between gap-3 flex-wrap">
            <h2
              style={{
                ...serif,
                fontWeight: 600,
                fontSize: 17,
                color: C.ink,
                marginBottom: 4,
              }}
            >
              Propositions des participants
            </h2>

            <span
              style={{
                ...mono,
                fontSize: 9.5,
                color: C.graphite,
                border: `1px solid ${C.line}`,
                padding: "3px 7px",
              }}
            >
              {adminProposals.length} proposition
              {adminProposals.length > 1 ? "s" : ""}
            </span>
          </div>

          <p
            style={{
              fontSize: 12.5,
              color: C.graphite,
              marginBottom: 14,
            }}
          >
            Propositions enregistrées par les participants.
          </p>

          {adminProposals.length === 0 ? (
            <div style={{ color: C.graphite, fontSize: 13 }}>
              Aucune proposition pour l'instant.
            </div>
          ) : (
            <div className="flex flex-col gap-3">
              {adminProposals.map((proposal) => {
                const author =
                  participantsById.get(proposal.participant_id)?.name ||
                  "Participant inconnu";

                return (
                  <div
                    key={proposal.id}
                    className="p-4"
                    style={{
                      border: `1px solid ${C.line}`,
                      background: "#F5F3EE",
                    }}
                  >
                    <div className="flex items-start justify-between gap-4">
                      <div className="min-w-0">
                        <div
                          style={{
                            ...mono,
                            fontSize: 9.5,
                            color: C.brass,
                            letterSpacing: 1,
                            marginBottom: 4,
                          }}
                        >
                          {proposal.category?.toUpperCase() || "PROPOSITION"}
                        </div>

                        <div
                          style={{
                            ...serif,
                            fontSize: 16,
                            fontWeight: 600,
                            color: C.ink,
                          }}
                        >
                          {proposal.title}
                        </div>
                      </div>

                      <button
                        type="button"
                        onClick={() => deleteProposal(proposal.id)}
                        className="shrink-0 px-3 py-2"
                        style={{
                          border: `1px solid ${C.line}`,
                          color: C.ink,
                          background: "transparent",
                          fontSize: 11,
                          ...mono,
                          cursor: "pointer",
                        }}
                      >
                        <Trash2 size={13} className="inline mr-1" />
                        Supprimer
                      </button>
                    </div>

                    {proposal.description && (
                      <div
                        style={{
                          fontSize: 12.5,
                          color: C.graphite,
                          marginTop: 9,
                          lineHeight: 1.5,
                        }}
                      >
                        {proposal.description}
                      </div>
                    )}

                    <div
                      className="flex flex-wrap gap-3"
                      style={{
                        ...mono,
                        fontSize: 9.5,
                        color: C.graphite,
                        marginTop: 10,
                      }}
                    >
                      <span>Par {author}</span>

                      {proposal.location && (
                        <span>📍 {proposal.location}</span>
                      )}

                      {proposal.price !== null &&
                        proposal.price !== undefined && (
                          <span>{proposal.price} €</span>
                        )}

                      <span>
                        {proposal.created_at
                          ? fmtTime(new Date(proposal.created_at).getTime())
                          : "Date inconnue"}
                      </span>
                    </div>
                  </div>
                );
              })}
            </div>
          )}
        </div>

        <div style={{ background: C.bone, border: `1px solid ${C.line}`, boxShadow: `6px 6px 0 rgba(11,15,20,0.14)` }} className="p-5">
          <h2 style={{ ...serif, fontWeight: 600, fontSize: 17, color: C.ink, marginBottom: 10 }}>Journal complet ({logs.length})</h2>
          <div className="flex flex-col gap-1.5 max-h-96 overflow-y-auto">
            {logs.length === 0 ? (
              <div style={{ color: C.graphite, fontSize: 13 }}>Aucune activité enregistrée.</div>
            ) : (
              logs.map((l) => (
                <div key={l.id} style={{ fontSize: 12, ...mono, color: C.ink, opacity: 0.85 }}>
                  <span style={{ color: C.brass }}>{fmtTime(l.at)}</span> — {LOG_LABELS[l.type] ? LOG_LABELS[l.type](l) : l.type}
                </div>
              ))
            )}
          </div>
        </div>
      </div>
    </div>
  );
}

// ---------- Dossier complet ----------
const PROGRAMME = [
  { jour: "Lun 13/03", matin: "Trajet Eurostar", apresmidi: "Installation + Tower Bridge / South Bank (gratuit)", soir: "Repas Tesco meal deal" },
  { jour: "Mar 14/03", matin: "Old Bailey — galerie publique (10h-12h40)", apresmidi: "British Museum — gratuit", soir: "Repas Tesco meal deal" },
  { jour: "Mer 15/03", matin: "HMS Belfast — 20-24 £", apresmidi: "Westminster : Big Ben, Parlement, Abbaye", soir: "Restaurant + sortie encadrée par un professeur" },
  { jour: "Jeu 16/03", matin: "Imperial War Museum — gratuit", apresmidi: "Caserne de pompiers (sous réserve) + Buckingham Palace", soir: "Repas Tesco meal deal" },
  { jour: "Ven 17/03", matin: "National Gallery ou Camden Market", apresmidi: "Trajet retour Eurostar", soir: "—" },
];

const BUDGET = [
  { poste: "Eurostar A/R", detail: "Réservation ~ janvier 2028", cout: "140–260 €" },
  { poste: "Hébergement", detail: "Clink261, dortoir ou chambre 4, 4 nuits", cout: "70–140 €" },
  { poste: "Transport local", detail: "Travelcard zones 1-2, 5 jours", cout: "35–40 €" },
  { poste: "Repas", detail: "Tesco meal deals + 1 restaurant", cout: "46–49 €" },
  { poste: "HMS Belfast", detail: "Entrée", cout: "23–28 €" },
  { poste: "Assurance voyage", detail: "RC + rapatriement (obligatoire)", cout: "10–15 €" },
];

const CONTACTS = [
  { label: "Réservation groupes Eurostar", value: "eurostar.com (espace groupes) — 0344 822 5822" },
  { label: "Old Bailey (pack pédagogique)", value: "info@oldbaileyinsight.co.uk" },
  { label: "Hébergement", value: "directement sur le site de chaque établissement, option « groups / schools »" },
  { label: "Visite caserne de pompiers", value: "london-fire.gov.uk — rubrique « Book your school visit »" },
];

function DossierTable({ title, children }) {
  return (
    <div style={{ background: "#E8E6E0", border: `1px solid ${C.line}`, boxShadow: `6px 6px 0 rgba(11,15,20,0.14)` }} className="p-6">
      <h2 style={{ ...serif, fontWeight: 600, fontSize: 19, color: C.ink, marginBottom: 14 }}>{title}</h2>
      {children}
    </div>
  );
}

function Dossier() {
  return (
    <div className="flex flex-col gap-7 max-w-4xl mx-auto pb-14">
      <DossierTable title="Programme jour par jour">
        <div className="flex flex-col divide-y" style={{ borderColor: C.line }}>
          {PROGRAMME.map((p) => (
            <div key={p.jour} className="py-3 grid grid-cols-1 md:grid-cols-4 gap-1.5">
              <div style={{ ...mono, fontSize: 12, color: C.brass, fontWeight: 600 }}>{p.jour}</div>
              <div style={{ fontSize: 13, color: C.ink }}>{p.matin}</div>
              <div style={{ fontSize: 13, color: C.ink }}>{p.apresmidi}</div>
              <div style={{ fontSize: 13, color: C.ink, opacity: 0.75 }}>{p.soir}</div>
            </div>
          ))}
        </div>
      </DossierTable>

      <DossierTable title="Budget estimé par personne">
        <div className="flex flex-col divide-y" style={{ borderColor: C.line }}>
          {BUDGET.map((b) => (
            <div key={b.poste} className="py-2.5 flex items-center justify-between gap-3">
              <div>
                <div style={{ fontSize: 14, color: C.ink, fontWeight: 600 }}>{b.poste}</div>
                <div style={{ fontSize: 12, color: C.graphite }}>{b.detail}</div>
              </div>
              <div style={{ ...mono, fontSize: 13, color: C.ink, whiteSpace: "nowrap" }}>{b.cout}</div>
            </div>
          ))}
        </div>
        <div className="mt-4 pt-3 flex items-center justify-between" style={{ borderTop: `2px dashed ${C.line}` }}>
          <span style={{ ...serif, fontWeight: 600, fontSize: 16, color: C.ink }}>Total estimé</span>
          <span style={{ ...serif, fontWeight: 600, fontSize: 20, color: C.signal }}>≈ 325 à 530 €</span>
        </div>
        <p style={{ fontSize: 11.5, color: C.graphite, marginTop: 8 }}>
          La carte européenne d'assurance maladie (GHIC) ne suffit pas seule : elle ne couvre ni le rapatriement, ni la responsabilité civile, ni les bagages. Une assurance extra-scolaire est fortement recommandée en plus.
        </p>
      </DossierTable>

      <DossierTable title="Contacts utiles">
        <div className="flex flex-col gap-2.5">
          {CONTACTS.map((c) => (
            <div key={c.label} style={{ fontSize: 13 }}>
              <span style={{ ...mono, color: C.brass, fontWeight: 500 }}>{c.label} — </span>
              <span style={{ color: C.ink }}>{c.value}</span>
            </div>
          ))}
        </div>
      </DossierTable>

      <DossierTable title="Sorties du soir">
        <ul className="flex flex-col gap-2" style={{ fontSize: 13, color: C.ink, listStyle: "disc", paddingLeft: 18 }}>
          <li>Sortie encadrée par un professeur : toujours autorisée, quel que soit l'âge.</li>
          <li>Élèves mineurs : pas d'autonomie en solo le soir, pas d'alcool/tabac, boîtes de nuit interdites.</li>
          <li>Élèves majeurs : autorisés par la loi britannique, mais restent soumis au règlement intérieur de l'établissement.</li>
          <li>Les règles précises d'autonomie pour les majeurs seront validées par l'équipe pédagogique avant le départ.</li>
        </ul>
      </DossierTable>
    </div>
  );
}

// ---------- App ----------
export default function App() {
  const [tab, setTab] = useState("hebergement");
  const [voterName, setVoterName] = useState(null);
  const [needName, setNeedName] = useState(true);
  const [data, setData] = useState(emptyTripData());
  const [showAdminGate, setShowAdminGate] = useState(false);
  const [showAdmin, setShowAdmin] = useState(false);
  const [isAdmin, setIsAdmin] = useState(false);

  // Identité Supabase de la session actuelle.
  const [supabaseUser, setSupabaseUser] = useState(null);
  const [participant, setParticipant] = useState(null);
  const [supabaseReady, setSupabaseReady] = useState(false);
  const [dbOptions, setDbOptions] = useState([]);

  const dataRef = useRef(data);
  useEffect(() => {
    dataRef.current = data;
  }, [data]);

  const hasStorage = typeof window !== "undefined" && !!window.storage;

  const initSupabaseSession = useCallback(async () => {
    try {
      let { data: sessionData, error: sessionError } =
        await supabase.auth.getSession();

      if (sessionError) {
        console.error("Supabase session:", sessionError);
        return;
      }

      let session = sessionData?.session;

      // Aucun compte connecté : création d'une session anonyme.
      if (!session) {
        const { data: anonymousData, error: anonymousError } =
          await supabase.auth.signInAnonymously();

        if (anonymousError) {
          console.error("Supabase anonymous sign-in:", anonymousError);
          return;
        }

        session = anonymousData?.session;
      }

      const user = session?.user;

      if (!user) {
        console.error("Aucun utilisateur Supabase disponible.");
        return;
      }

      setSupabaseUser(user);

      const { data: existingParticipant, error: participantError } =
        await supabase
          .from("participants")
          .select("*")
          .eq("user_id", user.id)
          .maybeSingle();

      if (participantError) {
        console.error("Lecture participant:", participantError);
        return;
      }

      if (existingParticipant) {
        setParticipant(existingParticipant);

        if (existingParticipant.name) {
          setVoterName(existingParticipant.name);
          setNeedName(false);
        }
      }

      setSupabaseReady(true);
    } catch (error) {
      console.error("Initialisation Supabase:", error);
    }
  }, []);


  useEffect(() => {
    initSupabaseSession();
  }, [initSupabaseSession]);

  useEffect(() => {
    if (!supabaseReady) return;

    let cancelled = false;

    const loadDbOptions = async () => {
      const { data: options, error } = await supabase
        .from("options")
        .select("*")
        .order("created_at", { ascending: true });

      if (cancelled) return;

      if (error) {
        console.error("Lecture options Supabase:", error);
        return;
      }

      setDbOptions(options || []);
    };

    loadDbOptions();

    const interval = setInterval(loadDbOptions, 5000);

    return () => {
      cancelled = true;
      clearInterval(interval);
    };
  }, [supabaseReady]);

  const fetchAndApply = useCallback(async () => {
    if (!hasStorage) return;
    try {
      const res = await window.storage.get("trip-data", true);
      if (res?.value) {
        const parsed = normalizeData(JSON.parse(res.value));
        setData(parsed);
        dataRef.current = parsed;
      }
    } catch {
      // pas encore de données partagées
    }
  }, [hasStorage]);

  useEffect(() => {
    if (!hasStorage) return;
    let cancelled = false;
    window.storage
      .get("voter-name", false)
      .then((res) => {
        if (!cancelled && res?.value) {
          setVoterName(res.value);
          setNeedName(false);
        }
      })
      .catch(() => {});
    fetchAndApply();
    return () => {
      cancelled = true;
    };
  }, [hasStorage, fetchAndApply]);

  useEffect(() => {
    if (!hasStorage) return;
    const interval = setInterval(fetchAndApply, 6000);
    return () => clearInterval(interval);
  }, [hasStorage, fetchAndApply]);

  const mutateTripData = useCallback(
    async (mutator) => {
      let base = dataRef.current;
      if (hasStorage) {
        try {
          const res = await window.storage.get("trip-data", true);
          if (res?.value) base = normalizeData(JSON.parse(res.value));
        } catch {
          // rien en base, on part de l'état local
        }
      }
      const next = mutator(base);
      setData(next);
      dataRef.current = next;
      if (hasStorage) {
        try {
          await window.storage.set("trip-data", JSON.stringify(next), true);
        } catch (e) {
          console.error("Erreur de sauvegarde", e);
        }
      }
    },
    [hasStorage]
  );

  const submitName = async (name) => {
    const cleanName = name.trim();
    if (!cleanName) return;

    setVoterName(cleanName);
    setNeedName(false);

    // Compatibilité temporaire avec l'ancien stockage.
    if (hasStorage) {
      try {
        await window.storage.set("voter-name", cleanName, false);
      } catch {}
    }

    // Enregistrement du participant dans Supabase.
    if (supabaseUser) {
      const now = new Date().toISOString();

      const { data: savedParticipant, error: participantError } =
        await supabase
          .from("participants")
          .upsert(
            {
              user_id: supabaseUser.id,
              name: cleanName,
              last_seen: now,
            },
            { onConflict: "user_id" }
          )
          .select("*")
          .single();

      if (participantError) {
        console.error("Erreur participant Supabase:", participantError);
      } else {
        setParticipant(savedParticipant);

        // Chaque validation du nom correspond à une connexion enregistrée.
        const { error: logError } = await supabase
          .from("activity_logs")
          .insert({
            participant_id: savedParticipant.id,
            action: "join",
            details: {
              name: cleanName,
            },
          });

        if (logError) {
          console.error("Erreur journal Supabase:", logError);
        }
      }
    }

    // Ancien journal conservé temporairement pendant la migration.
    mutateTripData((base) => ({
      ...base,
      logs: [
        ...(base.logs || []),
        {
          id: Date.now() + Math.random(),
          type: "join",
          name: cleanName,
          at: Date.now(),
        },
      ].slice(-300),
    }));
  };
  const updateParticipantPresence = useCallback(async () => {
    if (!supabaseUser) return;

    const now = new Date().toISOString();

    const { data: updatedParticipant, error } = await supabase
      .from("participants")
      .update({ last_seen: now })
      .eq("user_id", supabaseUser.id)
      .select("*")
      .maybeSingle();

    if (error) {
      console.error("Mise à jour participant:", error);
      return;
    }

    if (updatedParticipant) {
      setParticipant(updatedParticipant);
    }
  }, [supabaseUser]);

  const skipName = () => setNeedName(false);

  const toggleVote = async (category, itemId, itemName, choice) => {
    if (!voterName || !participant || !supabaseUser || !supabaseReady) return;

    const dbCategory = DB_CATEGORY_BY_UI[category];

    if (!dbCategory) {
      console.error("Catégorie inconnue :", category);
      return;
    }

    const dbOption = dbOptions.find(
      (option) =>
        option.title === itemName &&
        option.category === dbCategory
    );

    if (!dbOption) {
      console.error("Option Supabase introuvable :", {
        category,
        itemName,
        dbCategory,
      });
      return;
    }

    const { data: existingVote, error: voteReadError } = await supabase
      .from("votes")
      .select("id, choice")
      .eq("participant_id", participant.id)
      .eq("option_id", dbOption.id)
      .maybeSingle();

    if (voteReadError) {
      console.error("Lecture vote Supabase:", voteReadError);
      return;
    }

    let action;

    if (existingVote && existingVote.choice === (choice === "oui")) {
      const { error: deleteError } = await supabase
        .from("votes")
        .delete()
        .eq("id", existingVote.id);

      if (deleteError) {
        console.error("Suppression vote Supabase:", deleteError);
        return;
      }

      action = "retiré son vote pour";
    } else {
      const now = new Date().toISOString();

      const { error: upsertError } = await supabase
        .from("votes")
        .upsert(
          {
            participant_id: participant.id,
            option_id: dbOption.id,
            choice: choice === "oui",
            updated_at: now,
          },
          { onConflict: "participant_id,option_id" }
        );

      if (upsertError) {
        console.error("Enregistrement vote Supabase:", upsertError);
        return;
      }

      action = choice === "oui" ? "voté oui pour" : "voté non pour";
    }

    const { error: logError } = await supabase
      .from("activity_logs")
      .insert({
        participant_id: participant.id,
        action: "vote",
        details: {
          category,
          item_id: itemId,
          option_id: dbOption.id,
          item_name: itemName,
          choice,
          result: action,
        },
      });

    if (logError) {
      console.error("Journal vote Supabase:", logError);
    }

    // Mise à jour temporaire de l'ancien état local pour garder l'interface fluide.
    mutateTripData((base) => {
      const catVotes = { ...(base.votes?.[category] || {}) };
      const itemVotes = { ...(catVotes[itemId] || {}) };

      if (existingVote && existingVote.choice === (choice === "oui")) {
        delete itemVotes[voterName];
      } else {
        itemVotes[voterName] = choice;
      }

      catVotes[itemId] = itemVotes;

      const logs = [
        ...(base.logs || []),
        {
          id: Date.now() + Math.random(),
          type: "vote",
          name: voterName,
          category,
          itemId,
          itemName,
          action,
          at: Date.now(),
        },
      ].slice(-300);

      return {
        ...base,
        votes: {
          ...base.votes,
          [category]: catVotes,
        },
        logs,
      };
    });
  };

  const createProposal = async (form, category) => {
    if (!participant || !supabaseUser || !supabaseReady) {
      console.error("Impossible de créer la proposition : participant non prêt.");
      return;
    }

    const title = form.name.trim();

    if (!title) return;

    const rawPrice = form.price.trim();
    const normalizedPrice = rawPrice
      ? rawPrice.replace(",", ".").replace(/[^\\d.-]/g, "")
      : "";

    const numericPrice =
      normalizedPrice && !Number.isNaN(Number(normalizedPrice))
        ? Number(normalizedPrice)
        : null;

    const { data: proposal, error } = await supabase
      .from("proposals")
      .insert({
        participant_id: participant.id,
        category,
        title,
        description: form.desc.trim(),
        location: form.where.trim() || null,
        price: numericPrice,
      })
      .select("*")
      .single();

    if (error) {
      console.error("Création proposition Supabase:", error);
      window.alert("Impossible d'enregistrer la proposition.");
      return;
    }

    console.log("Proposition créée :", proposal);

    const { data: newOptions, error: optionsError } = await supabase
      .from("options")
      .select("*")
      .eq("created_by", participant.id)
      .eq("title", title)
      .eq("is_custom", true)
      .order("created_at", { ascending: false })
      .limit(1);

    if (optionsError) {
      console.error("Lecture nouvelle option:", optionsError);
    } else if (newOptions?.[0]) {
      setDbOptions((current) => {
        const exists = current.some(
          (option) => option.id === newOptions[0].id
        );

        return exists
          ? current
          : [...current, newOptions[0]];
      });
    }

    mutateTripData((base) => ({
      ...base,
      logs: [
        ...(base.logs || []),
        {
          id: Date.now() + Math.random(),
          type: "propose",
          name: voterName || "Anonyme",
          category,
          itemName: title,
          at: Date.now(),
        },
      ].slice(-300),
    }));
  };

  const addActivity = (form) => {
    createProposal(form, "activity");
  };

  const addMeal = (form) => {
    createProposal(form, "meal");
  };

  const findDbOption = useCallback(
    (item) => {
      if (!item || !dbOptions.length) return null;

      return (
        dbOptions.find(
          (option) =>
            option.title === item.name &&
            (
              (item.category === "hebergement" && option.category === "lodging") ||
              (item.category === "repas" && option.category === "meal") ||
              (item.category === "activites" && option.category === "activity") ||
              !item.category
            )
        ) || null
      );
    },
    [dbOptions]
  );

  const dbCustomActivities = dbOptions
    .filter((option) => option.category === "activity" && option.is_custom)
    .map((option) => ({
      id: option.id,
      name: option.title,
      price: option.price != null ? `${option.price} €` : "À préciser",
      where: option.location || "À préciser",
      desc: option.description || "",
      category: "activites",
    }));

  const dbCustomMeals = dbOptions
    .filter((option) => option.category === "meal" && option.is_custom)
    .map((option) => ({
      id: option.id,
      name: option.title,
      price: option.price != null ? `${option.price} €` : "À préciser",
      where: option.location || "À préciser",
      desc: option.description || "",
      category: "repas",
    }));

  const activitiesAll = [
    ...ACTIVITIES_CULTURE,
    ...ACTIVITIES_SECURITY,
    ...(data.customActivities || []),
    ...dbCustomActivities,
  ];

  const mealsAll = [
    ...MEALS,
    ...(data.customMeals || []),
    ...dbCustomMeals,
  ];

  const allItemsById = {};
  [...LODGING, ...mealsAll, ...activitiesAll].forEach((it) => (allItemsById[it.id] = it));

  const mySelections = VOTE_CATEGORIES.flatMap((cat) => {
    const list = cat === "activites" ? activitiesAll : cat === "hebergement" ? LODGING : mealsAll;
    return list.filter((it) => (data.votes?.[cat]?.[it.id] || {})[voterName] === "oui").map((it) => it.name);
  });

  const removeCustomItem = (kind, itemId, itemName) => {
    mutateTripData((base) => {
      const key = kind === "activite" ? "customActivities" : "customMeals";
      const next = { ...base, [key]: (base[key] || []).filter((it) => it.id !== itemId) };
      // on retire aussi les votes associés, l'item n'existant plus
      const cat = kind === "activite" ? "activites" : "repas";
      const catVotes = { ...(next.votes?.[cat] || {}) };
      delete catVotes[itemId];
      next.votes = { ...next.votes, [cat]: catVotes };
      next.logs = [...(base.logs || []), { id: Date.now() + Math.random(), type: "remove", name: voterName || "Anonyme", itemName, at: Date.now() }].slice(-300);
      return next;
    });
  };

  const handleAdminSubmit = async (email, password) => {
    const { data: authData, error: authError } = await supabase.auth.signInWithPassword({
      email,
      password,
    });

    if (authError || !authData?.user) {
      return false;
    }

    const { data: organizer, error: organizerError } = await supabase
      .from("organizers")
      .select("user_id")
      .eq("user_id", authData.user.id)
      .maybeSingle();

    if (organizerError || !organizer) {
      await supabase.auth.signOut();
      return false;
    }

    setShowAdminGate(false);
    setShowAdmin(true);
    setIsAdmin(true);
    return true;
  };

  return (
    <div className="min-h-screen" style={{ background: C.ink }}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,400;9..144,600;9..144,700&family=IBM+Plex+Mono:wght@400;500&display=swap');
        .ticket-notch { position:absolute; left:-9px; width:18px; height:18px; border-radius:9999px; background:${C.ink}; }
        .ticket-notch-top { top:-9px; }
        .ticket-notch-bottom { bottom:-9px; }
      `}</style>

      {needName && <NameGate onSubmit={submitName} onSkip={skipName} />}
      {showAdminGate && <AdminGate onSubmit={handleAdminSubmit} onClose={() => setShowAdminGate(false)} />}
      {showAdmin && (
        <AdminPanel
          data={data}
          allItems={allItemsById}
          onClose={() => setShowAdmin(false)}
          supabaseClient={supabase}
        />
      )}

      <header className="voyage-cover px-6 md:px-14 pt-12 pb-10">
        <div className="voyage-cover-top">
          <div className="voyage-kicker">
            <span className="voyage-kicker-number">VL / 2028 / 01</span>
            <span>FILIÈRE SÉCURITÉ · PRÉPARATION COLLECTIVE</span>
          </div>

          <button onClick={() => setShowAdminGate(true)} className="voyage-admin-button flex items-center gap-1.5 shrink-0">
            <Lock size={11} /> Organisateur
          </button>
        </div>

        <div className="voyage-cover-main">
          <div className="voyage-cover-title">
            <div className="voyage-cover-eyebrow">PASSENGER TRAVEL DOSSIER</div>
            <h1>Paris <span>—</span> Londres</h1>
            <div className="voyage-cover-subtitle">PARIS → LONDON · 13—17 MARS 2028</div>
          </div>

          <div className="voyage-seal">
            <span>VL</span>
            <small>2028</small>
          </div>

          <div className="hidden md:block voyage-cover-route">
            <RoutePath />
          </div>
        </div>

        <div className="voyage-cover-info">
          <span>EUROSTAR 9020</span>
          <i />
          <span>13 → 17 MARS 2028</span>
          <i />
          <span>4 NUITS</span>
          <i />
          <span>15 PAX MAX</span>
        </div>
        {voterName && (
          <div className="mt-5 inline-block px-3 py-1" style={{ border: `1px solid ${C.brass}`, color: C.brass, fontSize: 11.5, ...mono }}>
            PASSAGER : {voterName.toUpperCase()}
          </div>
        )}
      </header>

      <section className="voyage-timeline px-6 md:px-14" aria-label="Chronologie du voyage">
        <div className="voyage-timeline-heading">
          <span>ITINÉRAIRE</span>
          <span>13—17 MARS 2028</span>
        </div>

        <div className="voyage-timeline-track">
          {voyageTimeline.map((day, index) => (
            <div className={`voyage-timeline-day ${index === 0 || index === voyageTimeline.length - 1 ? "terminal" : ""}`} key={day.date}>
              <div className="voyage-timeline-marker">
                <span />
              </div>
              <div className="voyage-timeline-date">{day.date}</div>
              <div className="voyage-timeline-title">{day.title}</div>
              <div className="voyage-timeline-detail">{day.detail}</div>
            </div>
          ))}
        </div>
      </section>

      <nav className="voyage-tabs px-6 md:px-14" style={{ borderBottom: `1px solid ${C.lineOnInk}` }}>
        {TABS.map((t) => (
          <button
            key={t.key}
            onClick={() => setTab(t.key)}
            className={`voyage-tab ${tab === t.key ? "active" : ""}`}
          >
            <span className="voyage-tab-number">{t.chapter}</span>
            <span className="voyage-tab-content">
              <span className="voyage-tab-label">
                {t.key === "dossier" && <FileText size={14} />}
                {t.label}
              </span>
              <span className="voyage-tab-subtitle">{t.subtitle}</span>
            </span>
          </button>
        ))}
      </nav>

      <main className="px-6 md:px-14 py-10" style={{ background: C.ink }}>
        {tab === "dossier" ? (
          <Dossier />
        ) : (
          <div className="flex flex-col gap-5 max-w-4xl mx-auto pb-14">
            {tab === "hebergement" &&
              LODGING.map((item) => (
                <Ticket key={item.id} item={item} category="hebergement" itemVotes={data.votes.hebergement?.[item.id]} voterName={voterName} onVote={(choice) => toggleVote("hebergement", item.id, item.name, choice)} />
              ))}

            {tab === "repas" && (
              <>
                {mealsAll.map((item) => (
                  <Ticket
                    key={item.id}
                    item={item}
                    category="repas"
                    itemVotes={data.votes.repas?.[item.id]}
                    voterName={voterName}
                    onVote={(choice) => toggleVote("repas", item.id, item.name, choice)}
                    isAdmin={isAdmin}
                    onDelete={() => removeCustomItem("repas", item.id, item.name)}
                  />
                ))}
                <BlankTicket canOpen={!!voterName} onOpen={addMeal} label="Proposer un endroit pour manger" placeholderName="Nom du restaurant / de l'endroit" />
              </>
            )}

            {tab === "activites" && (
              <>
                <SectionTitle icon={<Landmark size={19} color={C.brass} />}>Culturel</SectionTitle>
                {ACTIVITIES_CULTURE.map((item) => (
                  <Ticket key={item.id} item={item} category="activites" itemVotes={data.votes.activites?.[item.id]} voterName={voterName} onVote={(choice) => toggleVote("activites", item.id, item.name, choice)} />
                ))}

                <SectionTitle icon={<span style={{ fontSize: 19 }}>🔐</span>}>Activités sécurité</SectionTitle>
                {ACTIVITIES_SECURITY.map((item) => (
                  <Ticket key={item.id} item={item} category="activites" itemVotes={data.votes.activites?.[item.id]} voterName={voterName} onVote={(choice) => toggleVote("activites", item.id, item.name, choice)} />
                ))}

                {(data.customActivities || []).length > 0 && <SectionTitle icon={<Plus size={19} color={C.brass} />}>Propositions du groupe</SectionTitle>}
                {(data.customActivities || []).map((item) => (
                  <Ticket
                    key={item.id}
                    item={item}
                    category="activites"
                    itemVotes={data.votes.activites?.[item.id]}
                    voterName={voterName}
                    onVote={(choice) => toggleVote("activites", item.id, item.name, choice)}
                    isAdmin={isAdmin}
                    onDelete={() => removeCustomItem("activite", item.id, item.name)}
                  />
                ))}
                <BlankTicket canOpen={!!voterName} onOpen={addActivity} label="Proposer une activité" placeholderName="Nom de l'activité" />
              </>
            )}
          </div>
        )}
      </main>

      {voterName && tab !== "dossier" && (
        <div className="fixed bottom-0 left-0 right-0" style={{ background: C.inkPanel, borderTop: `1.5px dashed ${C.lineOnInk}` }}>
          <div className="px-6 md:px-14 py-1.5 flex items-center gap-2 overflow-x-auto whitespace-nowrap">
            <span style={{ ...mono, color: C.brass, fontSize: 9.5, letterSpacing: 1.2 }}>TES OUI</span>
            {mySelections.length === 0 ? (
              <span style={{ color: C.bone, opacity: 0.5, fontSize: 11 }}>rien pour l'instant</span>
            ) : (
              mySelections.map((s, i) => (
                <span key={i} style={{ color: C.bone, fontSize: 11, ...mono, opacity: 0.85 }}>
                  {s}{i < mySelections.length - 1 ? "  ·" : ""}
                </span>
              ))
            )}
          </div>
        </div>
      )}
    </div>
  );
}

