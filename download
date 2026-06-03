function send(type,payload){postMessage(Object.assign({type:type},payload||{}));}
function decodeRtfHex(s){
  return s.replace(/\\'([0-9a-fA-F]{2})/g,function(_,h){
    var c=parseInt(h,16);
    if(c===0x96||c===0x97||c===0xAD) return '-';
    if(c===0xA3) return '£';
    if(c===0x92) return "'";
    if(c===0x93||c===0x94) return '"';
    return String.fromCharCode(c);
  });
}
function stripRtfFast(raw){
  var s=String(raw||'');
  if(s.indexOf('\\rtf')===-1 && s.indexOf('{\\rtf')===-1) return s;
  send('progress',{percent:10,message:'Reading RTF ledger text...'});
  s=decodeRtfHex(s);
  s=s.replace(/\\par[d]?/g,'\n').replace(/\\line/g,'\n').replace(/\\tab/g,' ');
  s=s.replace(/\\_[ ]?/g,'-');
  s=s.replace(/\\[a-zA-Z]+-?\d* ?/g,' ');
  s=s.replace(/\\\*/g,' ');
  s=s.replace(/[{}]/g,' ');
  s=s.replace(/\u8209\??/g,'-');
  s=s.replace(/[\u2010-\u2015\u2212]/g,'-');
  s=s.replace(/[ \t\f\v]+/g,' ');
  return s;
}
function moneyNum(s){
  s=String(s||'').replace(/[£, ]/g,'').replace(/[−–—]/g,'-');
  return /^-?\d+(?:\.\d{2})?$/.test(s)?Number(s):null;
}
function isMoneyLine(s){return /^-?£?\s*\d{1,3}(?:,\d{3})*\.\d{2}$|^-?£?\s*\d+\.\d{2}$/.test(String(s||'').trim());}
function ukDate(s){var m=String(s||'').match(/\b(\d{2})\/(\d{2})\/(\d{4})\b/);return m?m[3]+'-'+m[2]+'-'+m[1]:'';}
function isRef(s){return /^\d{5,}$/.test(String(s||'').trim());}
function isTrans(s){return /^\d{6,}\/\d{3}$/.test(String(s||'').trim());}
function parseLedger(raw){
  var text=stripRtfFast(raw);
  send('progress',{percent:30,message:'Splitting ledger into report lines...'});
  var lines=text.split(/\r?\n/).map(function(x){return x.trim().replace(/\s+/g,' ');}).filter(Boolean);
  var all=lines.join('\n');
  var clientMatch=all.match(/\n?([^\n]+?)\s*\((\d{6,})\)\s*\nUnpaid Bill Postings Only/i);
  var matterMatch=all.match(/\b(\d{8}\/\d{8})\b/);
  var client=clientMatch?clientMatch[1].trim():'';
  var matter=matterMatch?matterMatch[1]:'';
  var found=[];
  var seen={};
  send('progress',{percent:50,message:'Finding bill rows...'});
  for(var i=0;i<lines.length;i++){
    if(i%400===0) send('progress',{percent:50+Math.min(35,Math.round(i/Math.max(lines.length,1)*35)),message:'Scanning line '+i+' of '+lines.length+'...'});
    var d=ukDate(lines[i]);
    if(!d) continue;
    var type=(lines[i+1]||'').trim().toUpperCase();
    if(type!=='BC' && type!=='BILL' && type!=='BI') continue;
    var narr=(lines[i+2]||'').trim();
    if(!/bill|invoice/i.test(narr)) continue;
    var ref='';
    for(var r=i+3;r<Math.min(i+10,lines.length);r++){ if(isRef(lines[r])){ref=lines[r];break;} }
    var amount=0;
    for(var a=i-1;a>=Math.max(0,i-6);a--){ if(isMoneyLine(lines[a])){ var n=moneyNum(lines[a]); if(n!==null && n>0){amount=n;break;} } }
    var valsAfter=[];
    for(var j=i+3;j<Math.min(i+18,lines.length);j++){
      if(isMoneyLine(lines[j])){var v=moneyNum(lines[j]); if(v!==null && v>=0) valsAfter.push(v);}
    }
    var out=amount;
    if(valsAfter.length){
      var nonVat=valsAfter.filter(function(v){return Math.abs(v-amount)>0.009 && v>0;});
      var exact=valsAfter.filter(function(v){return Math.abs(v-amount)<0.009;});
      if(exact.length) out=exact[exact.length-1];
      else if(nonVat.length) out=nonVat[nonVat.length-1];
    }
    var key=[ref,d,amount,out].join('|');
    if((ref||amount) && !seen[key]){
      seen[key]=true;
      found.push({use:true,ref:ref,date:d,narr:narr,amount:amount,out:out});
    }
  }
  send('progress',{percent:95,message:'Finalising parsed results...'});
  return {client:client,matter:matter,bills:found,lineCount:lines.length};
}
onmessage=function(e){
  try{
    var result=parseLedger(e.data && e.data.raw || '');
    send('done',result);
  }catch(err){
    send('error',{message:err && err.message ? err.message : String(err)});
  }
};
