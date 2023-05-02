<div class="container">
         <h1 class="title">IZLO – Projekt 2: SMT solvery</h1>
                  <div class="row">
            <div class="col-xl-10"><h2 id="úvod">Úvod</h2>
<p>Poté, co si náš známý student z prvního projektu úspěšně naplánoval
předměty s využitím SAT solveru, dostal hlad a rozhodl se vyrazit na
oběd do menzy. Když dorazil na místo, byl velmi překvapen širokou
nabídkou jídel (především různých variant kuřecích plátků) a zmocnila se
ho rozhodovací paralýza. Nakonec usoudil, že jediným možným způsobem,
jak ideálně vybrat oběd, je aplikovat na tento problém logiku. Jelikož
už byl unaven kódováním problémů do výrokové logiky, rozhodl se zvolit
expresivnější prvořádovou logiku a využít SMT solver.</p>
<h2 id="zadání">Zadání</h2>
<p>Vstupem problému je pět jídel reprezentovaných čísly <span
class="math inline">1, 2,..., 5</span>, kde každé z jídel má
následující parametry nabývající hodnot z přirozených čísel (včetně
0):</p>
<ul>
<li>Cena <em>c<sub>i</sub></em>, kterou je potřeba
zaplatit za jednu porci daného jídlo.</li>
<li>Kalorická hodnota <em>k<sub>i</sub></em> jedné
porce daného jídla (čím vyšší, tím lepší).</li>
<li>Počet porcí <em>m<sub>i</sub></em> daného jídla
k dispozici.</li>
</ul>
<p>Cílem je najít vhodnou kombinaci jídel s co největší kalorickou
hodnotou, přičemž si student může objednat více porcí jednoho druhu
jídla. Tato kombinace musí splňovat následující podmínky:</p>
<ol type="1">
<li>Celková cena obědu je maximálně <em>(max_cena)</em>.</li>
<li>Požadovaný počet porcí každého jídla nepřesahuje maximální
počet.</li>
<li>Celkový součet kalorií je <em>maximální</em>, jinými slovy,
neexistuje jiné řešení, které by splňovalo podmínky <code>1</code> a
<code>2</code> a mělo vyšší celkový součet kalorií.</li>
</ol>
<h2 id="řešení">Řešení</h2>
<p>Vaším úkolem je do vyznačeného místa v souboru
<code>projekt2.smt2</code> doplnit formule, které zajistí, že:</p>
<ol type="1">
<li>Každá proměnná <em>n<sub>i</sub></em> je
interpretována jako objednaný počet porcí jídla <em>i</em>.</li>
<li>Proměnná <em>best</em> je interpretována
jako <em>optimální</em> kalorická hodnota.</li>
</ol>
<p>Tyto promměné přitom musí splňovat podmínky uvedené v předchozí
sekci.</p>
<p>Při řešení byste si měli vystačit s SMT-LIB příkazy
<code>declare-fun</code> (pro deklaraci proměnných) a
<code>assert</code> (pro přidání formulí, které mají být splněny). V
těchto formulích si vystačíte s booleovskými spojkami (<code>and</code>,
<code>or</code>, <code>not</code>, <code>=&gt;</code>, …),
kvantifikátory (<code>forall</code>, <code>exists</code>) a termy v
teorii celočíselné aritmetiky. Tato teorie obsahuje klasické funkce nad
čísly (<code>+</code>, <code>-</code>, <code>*</code>, <code>div</code>,
<code>mod</code>, <code>abs</code>) a predikáty pro porovnání čísel
(<code>=</code>, <code>&gt;</code>, <code>&lt;</code>,
<code>&lt;=</code>, <code>&gt;=</code>). Příklady použití lze nalézt
níže, případně na <a
href="https://moodle.vut.cz/mod/page/view.php?id=315108">stránce
předmětu</a>.</p>
<p>Referenčním (a doporučeným) SMT solverem je cvc5<a href="#fn1"
class="footnote-ref" id="fnref1" role="doc-noteref"><sup>1</sup></a>.
Pro řešení projektu zcela stačí použít jeho <a
href="https://cvc5.github.io/app/">online rozhraní</a>. Alternativně lze
solver stáhnout jako binárku z <a
href="https://github.com/cvc5/cvc5/releases/tag/cvc5-1.0.5">githubu</a>,
případně je k dispozici na serveru Merlin, kde ji lze použít pomocí
příkazu:</p>
<p><code>/usr/local/groups/verifikace/IZLO/cvc5-Linux projekt2.smt2</code></p>
<h2 id="formát-smt-lib">Formát SMT-LIB</h2>
<p>Formát SMT-LIB využívá prefixový zápis operátorů. Term
<code>x + y = x * y</code> je tedy zapsán jako
<code>(= (+ x y) (* x y))</code>. Při deklaraci proměnných je potřeba
uvést její tzv. <em>sort</em> (podobně jako typ proměnné například v
jazyce C), v tomto projektu budeme pracovat pouze s celočíselnými
proměnnými a odpovídajícím sortem <code>Int</code>.</p>
<pre class="smt2"><code>; Nastavení teorie, ve které má SMT solver pracovat.
; ALL značí všechny teorie podporované solverem
(set-logic ALL)

; Nastavení parametrů SMT solveru
(set-option :produce-models true)

; Deklarace celočíselný konstant jako nulární funkce)
(declare-fun x () Int)
(declare-fun y () Int)
(declare-fun z () Int)

; Deklarace binární funkce na celých číslech
(declare-fun c (Int Int) Int)

; Deklarace binárního predikátu na celých číslech
(declare-fun pred (Int Int) Bool)

; Přidání omezení reprezentující formuli (x * y = 0) → (x = 0 ∨ y = 0)
(assert
  (=&gt;
    (= (* x y) 0)
    (or (= x 0) (= y 0))
  )  
)

; Přidání dalšího omezení reprezentující formuli ∀a ∀b (∃c (a + b = c))
(assert
  (forall ((a Int) (b Int))
    (exists ((c Int))
      (= (+ a b) c)
    )
  )
)

; Ověření splnitelnost konjunkce všech omezení
(check-sat)

; Pokud je status sat, vypíše model
(get-model)

; Pokud je status sat, vypíše hodnoty termů x, y a x + y
(get-value (x y (+ x y)))
</code></pre>
<h2 id="kostra">Kostra</h2>
<p>Kostra projektu je ke stažení <a
href="https://www.fit.vutbr.cz/study/courses/IZLO/public/projekty/projekt2/projekt2.smt2">zde</a>.
Odevzdejte tento soubor, s řádkem “Zde doplnte vase reseni” nahrazeným
Vaším řešením. <strong>Pozor! Nemodifikujte nic jiného, jinak Vám
automatické testy zbytečně strhnou body.</strong></p>
<h2 id="testování">Testování</h2>
<p>Soubor s kostrou je doplněn několika testovacími vstupy, které
fungují následujícím způsobem. Každý test je rozdělen do dvou částí:</p>
<ul>
<li><strong>a)</strong> Oveří zda pro vstupní parametry existuje řešení
a vypíše jej. Očekávaný status je <code>sat</code>.</li>
<li><strong>b)</strong> Ověří zda pro vstupní parametry neexistuje jiné
řešení. Očekáváný status je <code>unsat</code>.</li>
</ul>
<p><strong>Skript samotný nekontroluje správnost výstupů</strong>.
Jednotlivé testy jsou implementované pomocí příkazu
<code>(check-sat-assuming (formulae))</code>, který ověří splnitelnost
globálních omezení (definovaných pomocí <code>assert</code>) v konjunci
s omezeními <code>formulae</code>. Pro debugování modelů v jednotlivých
testech lze využít příkaz <code>(get-value (terms))</code>.</p>
<h2 id="další-pokyny-a-doporučení">Další pokyny a doporučení</h2>
<ul>
<li>V případě špatného nebo příliš komplikovaného řešení se může stát,
že se solver „zasekne“. Při správném řešení by měl solver doběhnout
během několika sekund.</li>
<li>Odevzdáváte pouze soubor <code>projekt2.smt2</code> do IS VUT.</li>
<li>Své řešení vypracujte samostatně. Odevzdané projekty budou
kontrolovány proti plagiátorství, za nějž se považuje i sdílení
vlastního řešení s jinými lidmi.</li>
<li>Případné dotazy směřujte do fóra “Diskuze k projektům”.</li>
</ul>
</div>
         </div>
               </div>