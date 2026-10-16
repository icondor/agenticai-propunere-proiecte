# Proiecte pentru pauză: 20 de teme peste laburile 1-12

Aveți trei săptămâni de pauză. Alegeți **o temă** și lucrați la ea din când în când. Fiecare temă are trei niveluri. **Minim** se poate termina într-un weekend (cam 8-10 ore). **Bun** și **Avansat** sunt pentru cine are timp și chef.

Multe dintre aceste teme pot deveni baza proiectului final. **Proiectul final trebuie terminat până la ultimul lab, nu după.** O temă începută acum vă dă un avans real. Alegeți ceva care vă interesează.

## Restricție importantă

Proiectele folosesc **doar ce am făcut în laburile 1-12**. Fără persistence/HITL avansat (L13), fără multi-agent (L14-15) și fără LangSmith (L18). Le facem pe toate în sală. Dacă tema vă tentează spre ele, notați în README ce ați adăuga după acele laburi, dar nu le implementați acum.

HITL simplu, cu `humanInTheLoopMiddleware` și checkpointer în memorie, cum l-am făcut în L9, este permis.

## Ce am acoperit până acum

| Lab | Conținut |
|---|---|
| L1 | chat Next.js, streaming SSE, `llm.ts` cu mai mulți provideri, `cost.ts`, system prompt specializat |
| L2-3 | playground de prompturi, eval pe dataset, LLM-as-judge, fallback de provider cu `withFallbacks`, model local cu Ollama |
| L4 | memoria conversației: ConversationStore (memory, sqlite, postgres, redis), strategiile full, sliding, summary, hybrid |
| L5 | LCEL: RunnableSequence, RunnableParallel, RunnableBranch, RunnableLambda; triaj de tichete |
| L6 | structured output cu Zod, `tool()`, buclă manuală de tool calling, API-uri gratuite (open-meteo, frankfurter) |
| L7 | `createAgent`, pattern-ul ReAct, streaming cu `agent.stream` |
| L8 | recapitulare: workflows vs agents |
| L9 | middleware: pre-built (summarization, PII, toolCallLimit, HITL) și custom (audit, costGuard, anonimizare); `FakeToolCallingModel` pentru teste |
| L11 | RAG: extragere din PDF, chunking, embeddings, vector store propriu, eval cu hit rate și MRR |
| L12 | long-term memory cu Store, semantic search, LangGraph (StateGraph, conditional edges, reducers) |

## Cerințe comune, la orice temă și orice nivel

1. **README** cu: ce face proiectul, cum se rulează (o singură comandă), ce chei de API trebuie setate și ce costă o rulare.
2. **`llm.ts` comutabil**: providerul se schimbă dintr-o variabilă de mediu, nu din cod.
3. **Cel puțin un test offline** care rulează fără cheie de API, cu model fals sau embeddings false. Pentru agenți: `FakeToolCallingModel({ toolCalls })`, ca în L9. Pentru `withStructuredOutput`: `FakeToolCallingModel({ structuredResponse })`, ca în testele din L12. Pentru embeddings: clasa de embeddings false din L11 sau L12. Modelul fals întoarce mereu același răspuns, deci testul verifică codul din jurul modelului (reguli, rutare, parsare), nu calitatea modelului. Calitatea o măsoară setul de eval.
4. **Un set de eval** de minimum 15 exemple, cu rezultatul măsurat (acuratețe, hit rate, scor de judge, după caz).
5. **Un raport de cost**: câți tokeni și cât a costat rularea pe setul de eval, calculat cu `cost.ts`. `cost.ts` are prețuri doar pentru modelele de chat, deci costul embeddings îl estimați separat, din prețul providerului.
6. **Date sintetice sau publice.** Nu folosiți date personale reale: facturi, extrase de cont, emailuri, CV-uri ale unor oameni reali.
7. **Codul compilează**: `tsc --noEmit` trece fără erori.

Sfat pentru cost: dezvoltați pe Gemini free tier sau pe un model local cu Ollama și folosiți modelul plătit doar pentru rularea finală pe setul de eval.

---

# RAG și căutare

## 1. RAG peste transcrieri YouTube

Un asistent care răspunde la întrebări despre conținutul unui canal tematic (un curs, un podcast tehnic) și citează sursa cu link la minutul exact.

**Laburi:** L11, L5, L1

- **Minim:** 5-10 transcrieri ale aceluiași canal. Chunking pe segmente cu timestamp. RAG clasic ca lanț LCEL, care răspunde din consolă cu citare `[titlu video, mm:ss]`. Eval: 15 întrebări cu video-ul și intervalul corect.
- **Bun:** interfață de chat cu streaming. Citările sunt linkuri care pornesc video-ul la secunda potrivită (`&t=123s`). Răspunsul spune explicit «nu am găsit» când informația lipsește. Comparați chunking pe lungime fixă cu chunking pe segmente.
- **Avansat:** RAG agentic (retrieval ca tool) comparat cu RAG clasic pe același set de eval. Filtre pe metadate: video, dată, vorbitor.

**Atenție:** obținerea transcrierilor nu face parte din curs. Bibliotecile care le descarcă automat sunt neoficiale și se strică des, iar YouTube blochează adesea cererile venite din cloud. Varianta sigură: subtitrări descărcate manual de pe canale cu licență Creative Commons, salvate ca fișiere text. Transcrierea audio cu Whisper e posibilă, dar e un instrument nou, în afara cursului. Nu redistribuiți transcrierile.

## 2. Asistent pe regulamentul facultății sau al unei firme

Răspunde doar din documentele oficiale (regulament, ghid de onboarding, politici interne publice), cu citare, și refuză când răspunsul nu e în documente.

**Laburi:** L11, L9, L5

- **Minim:** 1-3 PDF-uri indexate. RAG cu citare de secțiune și pagină. Eval: 15 întrebări, dintre care cel puțin 4 negative (răspunsul nu există în documente) la care asistentul trebuie să spună că nu știe.
- **Bun:** chunking structural (pe secțiuni și articole), comparat cu chunking naiv prin hit rate și MRR. Varianta de RAG agentic (`createAgent` cu retrieval ca tool) cu `piiMiddleware`. Atenție: middleware-ul se aplică doar la `createAgent`. În lanțul LCEL clasic nu există middleware, acolo anonimizarea e un pas `RunnableLambda` înainte de model.
- **Avansat:** detectarea întrebărilor ambigue: asistentul cere o clarificare în loc să ghicească. Raport cu cazurile în care a greșit și de ce.

## 3. RAG cross-lingual

Documente publice în engleză (de exemplu legislație UE de pe EUR-Lex sau documentație tehnică), întrebări în română, răspunsuri în română cu citare din textul englez.

**Laburi:** L11

- **Minim:** 1 document indexat. 15 întrebări în română cu pasajul corect cunoscut. Hit rate și MRR.
- **Bun:** comparați două modele de embeddings și efectul `taskType` (RETRIEVAL_DOCUMENT / RETRIEVAL_QUERY) pe același set. Tabel cu rezultatele.
- **Avansat:** comparați trei strategii: întrebarea direct în română, întrebarea tradusă în engleză înainte de retrieval, ambele combinate. Concluzie argumentată: care merită costul extra.

Tema e mai mult de cercetare decât de aplicație. Potrivită pentru cine vrea să măsoare.

## 4. Generator de quiz și flashcards din notițe de curs

Primește notițe sau slide-uri și generează întrebări grilă cu explicații.

**Laburi:** L11, L6, L2-3

- **Minim:** pornind de la notițe în text sau Markdown. Structured output cu Zod: `{ intrebare, variante[4], corecta, explicatie, sursa }`. 10 întrebări generate dintr-un document, salvate în JSON.
- **Bun:** fiecare întrebare e verificată automat: răspunsul corect trebuie să se găsească în pasajul sursă (retrieval + verificare). Întrebările care nu trec sunt regenerate.
- **Avansat:** sursa poate fi și PDF (de exemplu exportul unui deck din curs), cu extragerea din L11. Extractorul din L11 e gândit pentru cărți, deci verificați întâi pe un singur PDF ce text iese. LLM-as-judge pe o grilă (claritate, o singură variantă corectă, distractori plauzibili). Interfață web în care parcurgi quiz-ul.

## 5. Memoria ședințelor

Din transcrierea unei ședințe scoate rezumatul și action items. Peste ședințele anterioare poți întreba «ce am decis despre X».

**Laburi:** L5, L6, L12

- **Minim:** structured output: `{ rezumat, decizii[], actionItems[{ ce, owner, deadline }] }` din 3 transcrieri sintetice. Lanț LCEL.
- **Bun:** deciziile sunt salvate în Store cu semantic search, namespace pe proiect. Întrebarea «ce am decis despre buget» găsește decizia din ședința de acum două săptămâni.
- **Avansat:** detectarea contradicțiilor: o decizie nouă care o contrazice pe una veche este semnalată. Eval: 15 întrebări peste 5 ședințe.

---

# LCEL și fluxuri deterministe

## 6. Router de emailuri pe mai multe domenii

Emailurile care intră sunt clasificate pe domeniu (suport, HR, juridic, vânzări) și fiecare primește un răspuns schiță cu promptul specializat al domeniului.

**Laburi:** L5, L6, L2-3

- **Minim:** clasificare cu structured output, apoi RunnableBranch cu câte un prompt pe domeniu. 20 de emailuri sintetice etichetate. Acuratețea clasificării.
- **Bun:** `batch` pe 50 de emailuri. Matrice de confuzie: ce domenii se confundă între ele. Fallback de provider cu `withFallbacks`, ca în `llm.ts`.
- **Avansat:** reguli în cod peste clasificarea modelului (de exemplu: orice email care conține «avocat» sau «instanță» merge la juridic). Comparați acuratețea fără și cu reguli.

## 7. Extractor de facturi și bonuri

Din text sau PDF extrage datele facturii, le validează și le exportă în CSV.

**Laburi:** L6, L2-3, L1

- **Minim:** facturi sintetice ca text. Schemă Zod: furnizor, CUI, dată, linii (produs, cantitate, preț), total. 15 facturi. Eval: câmpuri corecte față de valorile reale. Export în CSV, scris direct din cod.
- **Bun:** validare în cod: suma liniilor egală cu totalul, dată validă, CUI în format corect. Facturile care nu trec sunt marcate pentru verificare manuală, nu corectate în tăcere. Fallback de provider cu `withFallbacks`.
- **Avansat:** facturi ca PDF, citite cu `pdfjs-dist` ca în L11 (textul paginii, nu extractorul de cărți). Comparație între 2-3 provideri pe același set: acuratețe pe câmp, cost și latență.

## 8. Categorisirea unui extras de cont

Dintr-un CSV cu tranzacții, fiecare tranzacție primește o categorie (mâncare, transport, utilități, abonamente), apoi iese un raport lunar.

**Laburi:** L5, L6

- **Minim:** 50 de tranzacții sintetice, într-un CSV simplu generat de voi (fără virgule în câmpuri, ca să-l puteți citi cu `split`). Clasificare în `batch` cu structured output. Raport pe categorii în consolă.
- **Bun:** principiul din L12, «modelul extrage, codul decide»: regulile sigure (comerciant cunoscut, merge mereu la aceeași categorie) se aplică în cod, iar modelul primește doar tranzacțiile ambigue. Măsurați cât cost economisiți.
- **Avansat:** detectarea abonamentelor recurente și a cheltuielilor neobișnuite față de luna anterioară.

**Atenție:** doar date sintetice. Nu urcați extrasul vostru real de cont la un provider de LLM.

## 9. Code review pe un diff

Primește un diff și produce observații pe trei direcții: securitate, stil, teste.

**Laburi:** L5, L6, L2-3

- **Minim:** RunnableParallel cu trei recenzenți, apoi un pas care le combină. Observațiile ca structured output: `{ fisier, linie, severitate, problema, sugestie }`.
- **Bun:** eval pe 15 diff-uri cu bug-uri plantate de voi (SQL injection, lipsă validare, test lipsă). Câte bug-uri sunt prinse și câte observații false apar.
- **Avansat:** deduplicarea observațiilor care spun același lucru. Comparați un model mare cu unul mic pe același set.

## 10. Analiza review-urilor de clienți

Din câteva sute de review-uri iese un raport: teme principale, sentiment pe temă, exemple reprezentative.

**Laburi:** L5, L11, L6

- **Minim:** sentiment și temă extrase cu structured output, în `batch`. Raport cu numărul de review-uri pe temă.
- **Bun:** gruparea review-urilor asemănătoare cu embeddings și similaritatea cosinus din L11. Algoritm simplu: fiecare review intră în primul grup al cărui prim review îi seamănă peste un prag, altfel deschide un grup nou. Câte un review reprezentativ pe grup.
- **Avansat:** comparați două perioade: ce teme au crescut, ce teme au scăzut.

Surse posibile: seturi publice de review-uri (de exemplu de pe Kaggle) sau review-uri sintetice. Pentru Minim ajung 100-200 de review-uri, ca să nu plătiți mult.

## 11. Monitor de știri RSS cu rezumat zilnic

Citește câteva fluxuri RSS și produce un rezumat al zilei, fără duplicate, pe interesele utilizatorului. Scriptul se rulează manual; programarea automată nu face parte din temă.

**Laburi:** L5, L11, L12

- **Minim:** 3 fluxuri RSS, rezumat pe articol, rezumatul zilei într-un fișier Markdown. Citirea RSS e singurul element din afara cursului: folosiți o bibliotecă de parsare RSS din npm, sau salvați întâi câteva fluxuri ca fișiere și lucrați pe ele.
- **Bun:** deduplicare cu embeddings: aceeași știre de la două surse apare o singură dată. Interesele utilizatorului sunt salvate în Store și filtrează știrile.
- **Avansat:** feedback «mai mult de asta / mai puțin de asta», salvat în Store și folosit la rulările următoare.

**Atenție:** păstrați doar titlul, rezumatul vostru și linkul. Nu stocați și nu redistribuiți articolele întregi.

---

# Agenți și middleware

## 12. Agent SQL read-only

Un agent care răspunde la întrebări despre o bază de date, scriind și rulând interogări SELECT.

**Laburi:** L6, L7, L9

- **Minim:** bază SQLite publică (de exemplu Chinook), deschisă cu `better-sqlite3` ca în L4. Tools: `listeazaTabele`, `descrieTabel`, `ruleazaSelect`. `createAgent` care răspunde la 15 întrebări de business.
- **Bun:** middleware cu `wrapToolCall` care blochează orice interogare care nu e SELECT, `toolCallLimitMiddleware`, limită de rânduri întoarse. Eval: răspunsul agentului comparat cu rezultatul unei interogări scrise de voi.
- **Avansat:** când interogarea dă eroare, `createAgent` trimite automat mesajul de eroare înapoi modelului, care poate încerca din nou. Măsurați câte întrebări reușesc din prima și câte după corectare.

**Atenție:** conexiunea la baza de date trebuie deschisă read-only: `new Database(fisier, { readonly: true })`. Un filtru în prompt sau în middleware se poate ocoli, conexiunea read-only nu. Am verificat: pe o conexiune read-only, un DELETE dă eroarea «attempt to write a readonly database».

## 13. Agent de suport cu memorie pe client

Extinde demo11 din L12: agentul își amintește istoricul fiecărui client și detectează problemele recurente.

**Laburi:** L7, L9, L12

- **Minim:** `createAgent` cu Store și namespace pe client, ca în L12 Partea 1. La fiecare tichet nou, agentul vede tichetele anterioare ale clientului. 3 clienți sintetici.
- **Bun:** tools pentru status de serviciu și facturi (date fictive), ca în L12 Partea 2. `piiMiddleware`. Al treilea tichet pe aceeași temă primește automat prioritate mai mare (regula de recurență din demo11).
- **Avansat:** interfață web cu streaming (`agent.stream`, ca în L7), cu istoricul clientului vizibil lângă conversație.

## 14. Agent imobiliar peste o bază de anunțuri fictivă

Caută apartamente după criterii, le compară, calculează rata la bancă și propune vizionări.

**Laburi:** L6, L7, L9

- **Minim:** 30 de anunțuri fictive în JSON. Tools: `cautaAnunturi(filtre)`, `detaliiAnunt(id)`, `calculeazaRata(pret, avans, ani, dobanda)`. `createAgent`.
- **Bun:** programarea unei vizionări trece prin HITL din L9 (`humanInTheLoopMiddleware` înainte de tool-ul `programeazaVizionare`, aprobare în consolă, ca în demo4 din L9). `toolCallLimitMiddleware`.
- **Avansat:** structured output pentru comparația finală între 2-3 apartamente. Eval pe 15 cereri ale unor clienți fictivi.

## 15. Pachet de middleware reutilizabil

Un set de middleware custom, testate și documentate, pe care le poate folosi oricine în proiectul lui.

**Laburi:** L9

- **Minim:** două middleware custom cu `createMiddleware`: PII pentru România (CNP, IBAN, număr de telefon RO), pornind de la middleware-ul de anonimizare din L9, și costGuard pe utilizator. Teste cu `FakeToolCallingModel`, fără chei de API.
- **Bun:** încă două: audit log în SQLite (`better-sqlite3`, ca în L4) și rate limit pe utilizator (id-ul utilizatorului vine din `context`, ca în L12). README cu exemplu de folosire pentru fiecare.
- **Avansat:** test care verifică ordinea în care rulează hook-urile când sunt combinate toate patru. Documentați ce se întâmplă când două middleware modifică același mesaj.

Tema e pentru cine vrea calitate de cod și teste, nu un demo spectaculos.

## 16. Tutor personal cu memorie

Un tutor pentru un subiect ales de voi (TypeScript, SQL, o limbă străină) care își amintește nivelul și greșelile studentului.

**Laburi:** L12, L7, L9

- **Minim:** `createAgent` cu profilul studentului (nivel, obiective) în Store. Tutorul generează exerciții potrivite nivelului.
- **Bun:** greșelile sunt salvate ca memorii episodice, iar exercițiile următoare insistă pe ele. Semantic search pe greșeli.
- **Avansat:** `summarizationMiddleware` (L9) pentru sesiuni lungi, comparat cu sesiuni fără el: cost și calitate. Raport de progres pe ultimele 5 sesiuni.

---

# LangGraph

## 17. Flux de aprobare pentru cereri

Cereri de concediu sau de achiziții trec printr-un graf: clasificare, verificare de reguli, rutare pe praguri, notificări.

**Laburi:** L12 (Partea 2), L5

- **Minim:** StateGraph cu: clasificare (model), validare cu reguli în cod (de exemplu sumă peste 5000 merge la director), conditional edges, nod final. Jurnal de pași cu reducer.
- **Bun:** fan-out pentru notificări în paralel (solicitant, manager, contabilitate) și fan-in înainte de final. Diagrama Mermaid generată din graf, pusă în README.
- **Avansat:** aceeași logică scrisă și ca lanț LCEL. Comparație în README: ce a fost mai ușor de citit, de testat, de extins.

**Atenție:** aprobarea umană în mijlocul grafului (`interrupt`) se face în L13. Aici decizia umană poate fi un câmp din input, nu o pauză în graf.

## 18. Același agent scris în trei feluri

Un agent cu 2-3 tools (de exemplu vreme și curs valutar, cu API-uri gratuite), implementat de trei ori.

**Laburi:** L6, L7, L12

- **Minim:** varianta 1 cu buclă manuală de tool calling (L6) și varianta 2 cu `createAgent` (L7). Același set de 15 întrebări pe amândouă.
- **Bun:** varianta 3 ca StateGraph scris de mână, pornind de la ReAct-ul de mână din demo10 (L12). Tabel comparativ: tokeni, număr de pași, erori, linii de cod.
- **Avansat:** concluzie argumentată în README: când ați alege fiecare variantă și de ce. Cel mai didactic proiect din listă.

---

# Măsurători și comparații

## 19. Laborator pentru strategiile de memorie

O comparație măsurată între strategiile de memorie a conversației din L4.

**Laburi:** L4, L1, L2-3

- **Minim:** o conversație scriptată de 40 de mesaje, în care un fapt important apare la mesajul 3. Rulată dintr-un script cu `buildPrompt` din `lib/memory.ts` (L4), pe fiecare strategie: full, sliding, summary, hybrid. Cost pe strategie.
- **Bun:** 5 conversații și 10 întrebări de verificare de tipul «își mai amintește X?». Scor de memorare pe strategie, verificat cu LLM-as-judge.
- **Avansat:** tabel cost vs memorare și o recomandare: ce strategie pentru ce tip de aplicație. Un grafic e opțional; desenarea graficelor nu a fost în curs.

## 20. Benchmark de prompturi și modele pentru un task de business

Un task concret (de exemplu rezumarea unor contracte sau extragerea clauzelor de reziliere) evaluat pe mai multe prompturi și modele.

**Laburi:** L2-3, L1

- **Minim:** 2 prompturi x 2 modele, 15 documente. Grilă de judge cu 3 axe. Tabel cu scoruri.
- **Bun:** 3 prompturi x 3 modele, plus cost și latență. Verificați judge-ul: notați manual 5 rezultate și comparați cu notele lui.
- **Avansat:** testați dacă judge-ul favorizează modelul din aceeași familie cu el. Concluzie argumentată: ce combinație ați pune în producție.

---

## Teme pe care nu le recomand

- **Screening automat de CV-uri:** risc de discriminare și probleme GDPR.
- **Sfaturi medicale sau despre suplimente:** răspundere pentru recomandări greșite.
- **Scraping de site-uri** care interzic asta în termenii de utilizare.

Dacă totuși alegeți una dintre ele, folosiți doar date sintetice și adăugați în README o secțiune de riscuri.

## Ce predați la întoarcere

- linkul la repository;
- README cu nivelul atins (Minim, Bun sau Avansat) și rezultatele eval;
- un paragraf: ce n-a mers și ce ați face diferit.
