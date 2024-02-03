<script>
import Spinner from "./components/Spinner.svelte"
//"seehis", "shadow", "decides"
let loading=$state(false)
let inputBlocked=$state(false)
let solved=$state(false)
let solution=$state(null)
let anError=$state("")
let copied=$state("Share site with others!")


const ShowError=(err)=>{
    anError=err
    setTimeout(()=>{
        anError=""
    },3000)
}

const CopyLink=()=>{
    navigator.clipboard.writeText("https://www.carsonshort.com/summarysolver/")
    copied="Website link copied!"
    setTimeout(()=>{
        copied="Share site with others!"
    },1000)
}


let row1=$state({text:"", start:0, end:0})
let row2=$state({text:"", start:0, end:0})
let row3=$state({text:"", start:0, end:0})

const rowmap = {
  row1: row1,
  row2: row2,
  row3: row3,
};

const findCharMapping = async (str1, str2, str3) => {
    return new Promise((resolve, reject) => {
        const letterset = new Set();

        // Function to add string characters to the set
        const addToSet = (s) => {
            for (const x of s) {
                letterset.add(x);
            }
        };

        addToSet(str1);
        addToSet(str2);
        addToSet(str3);

        // Check if the total unique characters exceed 10
        if (letterset.size > 10) {
            console.log("Invalid strings");
            return reject("Too many different letters cannot map 1 letter to 1 integer");
        }

        // Function to convert string to integer based on character mapping
        const toInt = (s, charMap) => {
            let count = 0;
            let mult = 1;
            for (let i = s.length - 1; i >= 0; i--) {
                count += charMap[s[i]] * mult;
                mult *= 10;
            }
            return count;
        };

        // Generate all permutations of digits
        const generatePermutations = function* (arr, size) {
            const permute = function* (arr, index = 0) {
                if (index >= size) {
                    yield arr.slice(0, size);
                    return;
                }
                for (let i = index; i < arr.length; i++) {
                    [arr[index], arr[i]] = [arr[i], arr[index]];
                    yield* permute(arr, index + 1);
                    [arr[index], arr[i]] = [arr[i], arr[index]]; // backtrack
                }
            };
            yield* permute(arr);
        };

        const digits = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
        const letterArray = Array.from(letterset);
        for (const perm of generatePermutations(digits, letterset.size)) {
            const curMap = {};
            letterArray.forEach((letter, index) => {
                curMap[letter] = perm[index];
            });

            if (toInt(str1, curMap) + toInt(str2, curMap) === toInt(str3, curMap)) {
                console.log(curMap);
                return resolve(curMap);
            }
        }

        return reject("Went through all permutations and no solution was found");
    });
};

const checkSolution=async()=>{
    if (row1.text=="" || row2.text=="" || row3.text==""){
        document.getElementById("row1").focus()
        ShowError("all rows must be filled")
        return
    }


    if (loading==true || solved===true){return}
    loading=true
    inputBlocked=true
    setTimeout(()=>{
    findCharMapping(row1.text,row2.text,row3.text).then((sol)=>{
        solution=sol
        solved=true
    }).catch((err)=>{
        ShowError(err)
        inputBlocked=false
    }).finally(()=>{
        loading=false
    })
    },0)
}

const resetboard=()=>{
    solved=false
    row1.text=""
    row2.text=""
    row3.text=""
    inputBlocked=false;
    solution=null;

}



const handleSelect=(e)=>{
    rowmap[e.target.id].start=e.target.selectionStart
    rowmap[e.target.id].end=e.target.selectionEnd
}
const resetPosition=(e)=>{
    rowmap[e.target.id].start=e.target.value.length
    e.target.selectionStart=e.target.value.length
    rowmap[e.target.id].end=e.target.value.length
    e.target.selectionEnd=e.target.value.length
}

const handleChange=(e)=>{
    rowmap[e.target.id].start=e.target.selectionStart
    rowmap[e.target.id].end=e.target.selectionEnd
    rowmap[e.target.id].text=e.target.value.toUpperCase()
}

const handleKeyDown=(e)=>{
    rowmap[e.target.id].start=e.target.selectionStart
    rowmap[e.target.id].end=e.target.selectionEnd
    if (e.key=="Enter"){
        checkSolution()
    }
}





</script>

<section class="container">
    <div class="top">
        <p class="title"><a href="https://www.carsonshort.com">Carson Short</a> presents</p>
        <h1><a href="https://www.auftup.com/summary">Summary</a> Solver</h1>
        <h2 class="description">The solver for the daily number game that think's it's a word game.</h2>
        <p>The solver will return the first solution it finds if any are possible.</p>
    </div>
    <div class="letters" >
        <label  class="row start" aria="first row text entry">
            {#each row1.text.split("") as letter, index}
                {#if index<row1.end}
                    <span class={index>=row1.start&&index<row1.end? "letter-box selected" : "letter-box"}><span class={solution ? "letter moved" : "letter"}>{letter}</span>{#if solution} <span class="letter">{solution[letter]}</span> {/if}</span>
                {/if}
            {/each}
            {#if solution==null}
            <span class="letter-box"><input id="row1" onblur={resetPosition}  disabled={inputBlocked ? true : null} onkeyup={handleKeyDown} onclick={resetPosition} onselect={handleSelect} oninput={(e)=>{handleChange(e,row1.text)}} /></span>
            {/if}
            {#each row1.text.split("") as letter, index}
                {#if index>=row1.end}
                    <span class={index>=row1.start&&index<row1.end? "letter-box selected" : "letter-box"}><span class={solution ? "letter moved" : "letter"}>{letter}</span>{#if solution} <span class="letter">{solution[letter]}</span> {/if}</span>
                {/if}
            {/each}
        </label>
        <label class="row mid">
            <span class="letter-box plus"><span class="letter plus-letter">+</span></span>
            <span class="row">
            {#each row2.text.split("") as letter, index}
            {#if index<row2.end}
                <span class={index>=row2.start&&index<row2.end? "letter-box selected" : "letter-box"}><span class={solution ? "letter moved" : "letter"}>{letter}</span>{#if solution} <span class="letter">{solution[letter]}</span> {/if}</span>
            {/if}
            {/each}
            {#if solution==null}
            <span class="letter-box"><input id="row2" onblur={resetPosition}  disabled={inputBlocked ? true : null} onkeyup={handleKeyDown} onclick={resetPosition} onselect={handleSelect} oninput={(e)=>{handleChange(e,row2.text)}} /></span>
            {/if}
            
            {#each row2.text.split("") as letter, index}
                {#if index>=row2.end}
                    <span class={index>=row2.start&&index<row2.end? "letter-box selected" : "letter-box"}><span class={solution ? "letter moved" : "letter"}>{letter}</span>{#if solution} <span class="letter">{solution[letter]}</span> {/if}</span>
                {/if}
            {/each}
        </span>
        </label>
        <div class="linebreak"/>
        <label class="row end">
            {#each row3.text.split("") as letter, index}
            {#if index<row3.end}
                <span class={index>=row3.start&&index<row3.end? "letter-box selected" : "letter-box"}><span class={solution ? "letter moved" : "letter"}>{letter}</span>{#if solution} <span class="letter">{solution[letter]}</span> {/if}</span>
            {/if}
            {/each}
            {#if solution==null}
            <span class="letter-box"><input id="row3" onblur={resetPosition}  disabled={inputBlocked ? true : null} onkeyup={handleKeyDown} onclick={resetPosition} onselect={handleSelect} oninput={(e)=>{handleChange(e,row3.text)}} /></span>
            {/if}
            {#each row3.text.split("") as letter, index}
                {#if index>=row3.end}
                    <span class={index>=row3.start&&index<row3.end? "letter-box selected" : "letter-box"}><span class={solution ? "letter moved" : "letter"}>{letter}</span>{#if solution} <span class="letter">{solution[letter]}</span> {/if}</span>
                {/if}
            {/each}
        </label>
    </div>
    {#if solution==null}
    <button class="submit-button" onclick={checkSolution}>Submit</button>
    {/if}

    {#if anError}<div class="errorText">{anError}</div>{/if}

    {#if solved}
    <div class="complete">
        <h3>Solved!</h3>
        <br/>
        <p>
            The puzzle has been solved<br/>
            You can solve another or share this site with others.
            <br/><br/>
            <button onclick={copied.length>21?CopyLink:()=>{}}>{copied}</button>
            <br/>
            <br/>
            <button onclick={resetboard}>Reset Board</button>
        </p>
    </div>
    {/if}
    {#if loading}
    <Spinner/>
    {/if}

    <div class="bottom-message"><span>All credit for Summary game and styling goes to <a href="https://x.com/alizauf"> auf & tup</a>, check out summary at: <br/><a href="https://www.auftup.com/summary">auftup.com/summary</a></div>

</section>


<style>
    .submit-button{
        transition: 120ms;
        font-size:20px;
        font-weight:700;
        cursor:pointer;
        margin-top:20px;
        border:2px solid black;
        height:40px;
        color:white;
        background-color: black;
        width:200px;
        max-width:calc(100vw - 40px)
    }
    .submit-button:hover{
        background-color: white;
        color:black;
        
    }
    input::selection {
    background-color: transparent;
    color: transparent;
    }

    *{box-sizing: border-box;}

    :root{
        font-family: system-ui,-apple-system,BlinkMacSystemFont,Segoe UI,Roboto,Oxygen,Ubuntu,Cantarell,Open Sans,Helvetica Neue,sans-serif;
        box-sizing: border-box;
    }

    p,h1,h2,h3,h4,h5{
        padding: 0px;
        margin:0px;
    }
    .errorText{
        color:red;
    }
    h3{
        font-size:34px;
    }
    .complete button{
        font-size:20px;
        cursor:pointer;
        color:white;
        background-color: unset;
        border:unset;
        text-decoration: underline;

    }
    .complete button:hover{
        background-color: unset;
        border:unset;
    }
    .complete{
        max-width: calc(100vw - 20px);
        padding:20px;
        color:white;
        text-align: center;
        margin-top: 60px;
        background-color: black;
        border-radius:10px;
        min-height:300px;
        width:400px;
    }
    input{
        color:transparent;
        caret-color:black;
        margin:unset;
        padding:unset;
        border-color: transparent;
        height:56px;
        width:56px;
        padding-left:6px;
        background-color: transparent;
        font-size:40px;
    }
    .bottom-message{
        text-align: center;
        padding-bottom:20px;
        font-size: 12px;
        margin-top:100px;
        bottom:0px;
        width:100%;
        text-align: center;
    }
    .container{
        min-height:100vh;
        display: flex;
        flex-direction: column;
        align-items: center;
        width:100%;
    }
    .top{
        text-align: center;
        display: flex;
        align-items: center;
        flex-direction: column;
        max-width: 260px;
    }
    .title{
        font-weight: 700;
        font-size: 14px;
    }
    h1{
        text-transform: uppercase;
        font-size:37px;
        margin-bottom:2px;
    }
    .description{
        font-size:14px;
        margin-bottom:28px;
    }
    .letters{
        margin-top:60px;
        width:fit-content;
        display: flex;
        flex-direction: column;
        gap:5px;
    }
    .row{
        max-width:calc(100vw - 100px);
        min-width: 230px;
        justify-content: end;
        display: flex;
        gap:5px;
    }
    .mid{
        width:100%;
        justify-content: space-between;
    }
    .mid .row{
        min-width: unset;
    }
    .letter-box{
        display: flex;
        align-items: center;
        justify-content: center;
        text-align: center;
        border:1px solid black;
        width:60px;
        height:60px;
        
    }
    .selected{
        border-color:#007aff;
        border-width: 2px;
    }
    .letter{
        transition:500ms;
        font-size:27px;
        font-weight:700;
        text-transform: uppercase;
        /*transform:translate(-14px,-14px) scale(.5);*/
        position: absolute;
    }
    .moved{
        transform:translate(-20px, -20px) scale(.6);
    }
    .plus-letter{
        transform:unset;
    }
 

    .plus{
        border:2px solid rgba(255, 255, 255, 0)
    }

    .linebreak{
        width:100%;
        height:1px;
        background-color: black;

    }

    a{
        color:unset;
        text-decoration: unset;
    }
    a:hover{
        background: linear-gradient(60deg,#06b000,#17ff68 15%,#004dff 97%,#707070 100%,#0091ff);
        background-clip: text;
        -webkit-background-clip:text;
        -webkit-text-fill-color:transparent;
        color:transparent;
    }

    @media(max-width:768px){
        .letter-box{
            height:32px;
            width:32px;
        }
        .letter{
            font-size: 18px;
        }
        .moved{
        transform:translate(-10px, -10px) scale(.6);
        }   
        input{height:28px;
            width:28px;
        }
    }
</style>