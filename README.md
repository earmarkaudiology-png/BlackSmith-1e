var StrongholdSmithy = StrongholdSmithy || (function () {
    'use strict';

    var SCRIPT = 'StrongholdSmithy';
    var VERSION = '0.3.0-repairs';
    var STATE = 'STRONGHOLD_SMITHY';

    var CONFIG = {
        coinAttrs: { gp: 'gp', sp: 'sp', cp: 'cp' },
        reactionAttr: 'reactionbonus',
        monthlyBlacksmithGp: 30,
        helperMonthlyGp: {
            leatherworker: 3,
            tailor: 3,
            woodworker: 2
        },
        race: {
            human: { efficiency: 1, skillBonus: 0 },
            dwarf: { efficiency: 3, skillBonus: 25 },
            gnome: { efficiency: 2, skillBonus: 10 }
        },
        repairNotRepairablePct: 20,
        repairCostMinPct: 10,
        repairCostMaxPct: 90,
        colors: {
            panelBg: '#111111',
            panelBorder: '#666666',
            text: '#f0f0f0',
            subtext: '#cccccc',
            primaryBg: '#1d5f9b',
            primaryBorder: '#53a3e0',
            reportBg: '#245b2a',
            reportBorder: '#56b862',
            warnBg: '#9b6b00',
            warnBorder: '#d6b24d',
            dangerBg: '#8b1d1d',
            dangerBorder: '#d96a6a',
            buttonBg: '#000000',
            buttonBorder: '#ff2a00',
            buttonText: '#ff4500'
        },
        randomNames: [
            'Aldric', 'Bennit', 'Corvin', 'Darrik', 'Edric', 'Farlan', 'Garron', 'Halwen', 'Ivor', 'Joric',
            'Kellin', 'Loras', 'Merric', 'Naren', 'Orin', 'Perrin', 'Quint', 'Roder', 'Soren', 'Tarin',
            'Ulric', 'Varric', 'Wendell', 'Yorik', 'Zaren', 'Brom', 'Dain', 'Hob', 'Milo', 'Thane'
        ],
        haggleFailComments: [
            'That is my price, and it is a fair one.',
            'You are pressing too hard for too little.',
            'I have apprentices to feed and coal to buy.',
            'The iron does not shape itself for free.',
            'You can try elsewhere if the price bites.',
            'Good work costs good coin.',
            'I will not be talked down on honest labor.',
            'If you want cheap, buy poor work.',
            'My forge stays hot whether you pay or not.',
            'That is already closer than I like to cut it.',
            'I have better buyers than beggars.',
            'A careful job takes time and money.',
            'You are haggling with the wrong smith today.'
        ],
        products: {
            arrowheads: { label: 'Arrow Heads / Quarrel Tips', quantityBase: 30, baseDays: 30, basePriceGp: 2, skillMin: 0, category: 'other', support: [], leather: false, notes: 'Basic smith work.' },
            spearheads: { label: 'Spear Heads', quantityBase: 10, baseDays: 30, basePriceGp: 10, skillMin: 0, category: 'other', support: [], leather: false, notes: 'Basic smith work.' },
            morningstars: { label: 'Morning Stars', quantityBase: 5, baseDays: 30, basePriceGp: 25, skillMin: 0, category: 'other', support: [], leather: false, notes: 'Basic smith work.' },
            polearmheads: { label: 'Flails / Pole Arm Heads', quantityBase: 2, baseDays: 30, basePriceGp: 6, skillMin: 0, category: 'other', support: [], leather: false, notes: 'Basic smith work.' },
            ringmail: { label: 'Ring Mail', quantityBase: 1, baseDays: 20, basePriceGp: 30, skillMin: 1, category: 'metalarmor', support: ['leatherworker', 'tailor'], leather: true, notes: 'Requires leatherworker and tailor.' },
            scalemail: { label: 'Scale Mail', quantityBase: 1, baseDays: 30, basePriceGp: 45, skillMin: 1, category: 'metalarmor', support: ['leatherworker', 'tailor'], leather: true, notes: 'Requires leatherworker and tailor.' },
            studdedleather: { label: 'Studded Leather Armor', quantityBase: 1, baseDays: 15, basePriceGp: 15, skillMin: 1, category: 'other', support: ['leatherworker', 'tailor'], leather: true, notes: 'Requires leatherworker and tailor.' },
            splintedmail: { label: 'Splinted Mail', quantityBase: 1, baseDays: 20, basePriceGp: 80, skillMin: 2, category: 'metalarmor', support: ['leatherworker'], leather: true, notes: 'Requires leatherworker.' },
            chainmail: { label: 'Chain Mail', quantityBase: 1, baseDays: 45, basePriceGp: 75, skillMin: 3, category: 'metalarmor', support: [], leather: false, notes: 'Advanced armor work.' },
            bandedmail: { label: 'Banded Mail', quantityBase: 1, baseDays: 30, basePriceGp: 90, skillMin: 4, category: 'metalarmor', support: [], leather: false, notes: 'Master-level armor work.' },
            platemail: { label: 'Plate Mail', quantityBase: 1, baseDays: 90, basePriceGp: 400, skillMin: 4, category: 'metalarmor', support: [], leather: false, notes: 'Master-level armor work.' },
            helmetgreat: { label: 'Helmet, Great', quantityBase: 1, baseDays: 10, basePriceGp: 15, skillMin: 0, category: 'other', support: [], leather: false, notes: 'Standard smith work.' },
            helmetsmall: { label: 'Helmet, Small', quantityBase: 1, baseDays: 2, basePriceGp: 10, skillMin: 0, category: 'other', support: [], leather: false, notes: 'Standard smith work.' },
            leatherarmor: { label: 'Leather Armor', quantityBase: 1, baseDays: 10, basePriceGp: 5, skillMin: 0, category: 'other', support: ['leatherworker'], leather: true, notes: 'Requires leatherworker and oil-boiling facilities.' },
            paddedarmor: { label: 'Padded Armor', quantityBase: 1, baseDays: 30, basePriceGp: 4, skillMin: 0, category: 'other', support: ['tailor'], leather: false, notes: 'Requires tailor.' },
            shieldlarge: { label: 'Shield, Large', quantityBase: 1, baseDays: 2, basePriceGp: 15, skillMin: 0, category: 'other', support: ['woodworker'], leather: false, notes: 'Requires woodworker.' },
            shieldsmall: { label: 'Shield, Small', quantityBase: 1, baseDays: 1, basePriceGp: 10, skillMin: 0, category: 'other', support: ['woodworker'], leather: false, notes: 'Requires woodworker.' }
        }
    };

    _.each(CONFIG.products, function (product, key) {
        product.key = key;
        product.isArmor = _.contains([
            'ringmail', 'scalemail', 'studdedleather', 'splintedmail', 'chainmail',
            'bandedmail', 'platemail', 'leatherarmor', 'paddedarmor'
        ], key);
        product.isRepairableArmor = product.isArmor;
    });

    function checkInstall() {
        state[STATE] = state[STATE] || {};
        state[STATE].version = VERSION;
        state[STATE].nextCraftsmanId = state[STATE].nextCraftsmanId || 1;
        state[STATE].nextQuoteId = state[STATE].nextQuoteId || 1;
        state[STATE].nextJobId = state[STATE].nextJobId || 1;
        state[STATE].nextInspectionId = state[STATE].nextInspectionId || 1;
        state[STATE].employers = state[STATE].employers || {};
        _.each(_.keys(state[STATE].employers), function (characterId) {
            getEmployerState(characterId);
        });
        log(SCRIPT + ' v' + VERSION + ' ready.');
    }

    function registerEventHandlers() {
        on('chat:message', handleInput);
    }

    function esc(s) {
        return String(s || '')
            .replace(/&/g, '&amp;').replace(/</g, '&lt;')
            .replace(/>/g, '&gt;').replace(/"/g, '&quot;')
            .replace(/'/g, '&#39;');
    }

    var whisperTarget = '';

    function send(html) {
        if (whisperTarget) {
            sendChat('', '/w "' + whisperTarget.replace(/"/g, '') + '" ' + html);
        } else {
            sendChat('', '/direct ' + html);
        }
    }

    function box(title, body) {
        return '<div style="background:' + CONFIG.colors.panelBg + ';border:1px solid ' + CONFIG.colors.panelBorder + ';padding:10px;max-width:520px;color:' + CONFIG.colors.text + ';">' +
            '<div style="font-size:20px;font-weight:bold;margin-bottom:8px;">' + esc(title) + '</div>' + body + '</div>';
    }

    function line(label, value) {
        return '<div style="margin:2px 0;"><b>' + esc(label) + ':</b> ' + value + '</div>';
    }

    function btn(label, command, bg, border, width) {
        return '<a style="display:inline-block;text-align:center;margin:4px 6px 4px 0;padding:6px 8px;min-width:' + (width || 110) +
            'px;color:' + CONFIG.colors.buttonText + ';background:' + CONFIG.colors.buttonBg + ';border:1px solid ' +
            CONFIG.colors.buttonBorder + ';text-decoration:none;" href="' + esc(command) + '">' + esc(label) + '</a>';
    }

    function btnReset(label, command, width) {
        return '<a style="display:inline-block;text-align:center;margin:4px 6px 4px 0;padding:6px 8px;min-width:' + (width || 110) +
            'px;color:white;background:' + CONFIG.colors.dangerBg + ';border:1px solid ' + CONFIG.colors.dangerBorder +
            ';text-decoration:none;" href="' + esc(command) + '">' + esc(label) + '</a>';
    }

    function sectionHeader(title) {
        return '<div style="font-weight:bold;margin:10px 0 4px 0;">' + esc(title) + '</div>';
    }

    function parseArgs(content) {
        var args = {};
        var s = String(content || '');
        var i = s.indexOf(' --');
        while (i !== -1) {
            i += 3;
            var keyEnd = s.indexOf('|', i);
            if (keyEnd === -1) break;
            var key = s.substring(i, keyEnd).toLowerCase();
            var next = s.indexOf(' --', keyEnd + 1);
            var val = (next === -1 ? s.substring(keyEnd + 1) : s.substring(keyEnd + 1, next)).trim();
            args[key] = val;
            i = next;
        }
        return args;
    }

    function getObjById(type, id) {
        if (!id) return null;
        return getObj(type, id) || null;
    }

    function getCharacterByTokenId(tokenId) {
        var token = getObjById('graphic', tokenId);
        if (!token) return null;
        var rep = token.get('represents');
        return rep ? getObjById('character', rep) : null;
    }

    function getSelectedCharacter(msg) {
        if (!msg.selected || !msg.selected.length) return null;
        return getCharacterByTokenId(msg.selected[0]._id);
    }

    function canControl(playerid, character) {
        if (!character) return false;
        if (playerIsGM(playerid)) return true;
        var controlled = String(character.get('controlledby') || '');
        if (!controlled) return false;
        return _.contains(controlled.split(','), 'all') || _.contains(controlled.split(','), playerid);
    }

    function findAttr(characterId, name) {
        return findObjs({ type: 'attribute', characterid: characterId, name: name }, { caseInsensitive: true })[0] || null;
    }

    function getAttr(characterId, name, defaultValue) {
        var attr = findAttr(characterId, name);
        if (!attr) return defaultValue;
        var cur = attr.get('current');
        return cur === '' || cur === undefined || cur === null ? defaultValue : cur;
    }

    function setAttr(characterId, name, value) {
        var attr = findAttr(characterId, name);
        if (attr) attr.set('current', value);
        else createObj('attribute', { characterid: characterId, name: name, current: value });
    }

    function toNumber(value, defaultValue) {
        var n = Number(value);
        return isNaN(n) ? (defaultValue || 0) : n;
    }

    function parseMoneyCp(value) {
        return Math.round(toNumber(value, 0));
    }

    function toCp(gp, sp, cp) {
        return parseMoneyCp(gp) * 100 + parseMoneyCp(sp) * 10 + parseMoneyCp(cp);
    }

    function fromCp(totalCp) {
        totalCp = Math.max(0, Math.round(totalCp));
        var gp = Math.floor(totalCp / 100);
        totalCp -= gp * 100;
        var sp = Math.floor(totalCp / 10);
        totalCp -= sp * 10;
        return { gp: gp, sp: sp, cp: totalCp };
    }

    function formatCp(totalCp) {
        var c = fromCp(totalCp);
        var parts = [];
        if (c.gp) parts.push(c.gp + ' gp');
        if (c.sp) parts.push(c.sp + ' sp');
        if (c.cp || !parts.length) parts.push(c.cp + ' cp');
        return parts.join(', ');
    }

    function gpToCp(gp) {
        return Math.round(toNumber(gp, 0) * 100);
    }

    function percentOfCp(cp, pct) {
        var base = Number(cp) || 0;
        var percent = Number(pct) || 0;
        return Math.max(0, Math.round(base * percent / 100));
    }

    function getTreasuryCp(character) {
        return toCp(getAttr(character.id, CONFIG.coinAttrs.gp, 0), getAttr(character.id, CONFIG.coinAttrs.sp, 0), getAttr(character.id, CONFIG.coinAttrs.cp, 0));
    }

    function setTreasuryCp(character, totalCp) {
        var c = fromCp(totalCp);
        setAttr(character.id, CONFIG.coinAttrs.gp, c.gp);
        setAttr(character.id, CONFIG.coinAttrs.sp, c.sp);
        setAttr(character.id, CONFIG.coinAttrs.cp, c.cp);
    }

    function payCostOrReport(employer, totalCp) {
        var treasury = getTreasuryCp(employer);
        if (treasury < totalCp) return { ok: false, needed: totalCp, onHand: treasury };
        setTreasuryCp(employer, treasury - totalCp);
        return { ok: true, paid: totalCp };
    }

    function rollInteger(min, max) {
        return Math.floor(Math.random() * (max - min + 1)) + min;
    }

    function randomName() {
        return CONFIG.randomNames[rollInteger(0, CONFIG.randomNames.length - 1)];
    }

    function getEmployerState(characterId) {
        var e = state[STATE].employers[characterId] = state[STATE].employers[characterId] || {};
        e.craftsmen = e.craftsmen || [];
        e.quotes = e.quotes || [];
        e.jobs = e.jobs || [];
        e.inspections = e.inspections || [];
        _.each(e.quotes, function (q) {
            q.support = q.support || [];
            q.haggleAttempts = q.haggleAttempts || 0;
            q.haggleFailures = q.haggleFailures || 0;
            q.haggleDone = !!q.haggleDone || !!q.countered;
        });
        return e;
    }

    function resolveEmployer(msg, args) {
        var selected = getSelectedCharacter(msg);
        var explicit = getObjById('character', args.pc || args.employer);
        if (selected && canControl(msg.playerid, selected)) return selected;
        if (explicit && canControl(msg.playerid, explicit)) return explicit;
        return null;
    }

    function requireEmployer(msg, args) {
        var employer = resolveEmployer(msg, args);
        if (!employer) {
            send(box('Blacksmith', '<div>Select the buyer PC token first.</div>'));
            return null;
        }
        return employer;
    }

    function generateCraftsmanId() {
        return 'C' + (state[STATE].nextCraftsmanId++);
    }
    function generateQuoteId() {
        return 'Q' + (state[STATE].nextQuoteId++);
    }
    function generateJobId() {
        return 'J' + (state[STATE].nextJobId++);
    }
    function generateInspectionId() {
        return 'I' + (state[STATE].nextInspectionId++);
    }

    function getAllBlacksmiths() {
        return _.chain(state[STATE].employers || {}).keys().map(function (employerId) {
            var employerState = getEmployerState(employerId);
            var owner = getObjById('character', employerId);
            return _.map(_.filter(employerState.craftsmen, function (c) { return c && c.type === 'blacksmith'; }), function (c) {
                var copy = _.clone(c);
                copy.ownerId = employerId;
                copy.ownerName = owner ? (owner.get('name') || 'Employer') : 'Employer';
                return copy;
            });
        }).flatten().sortBy(function (c) { return ((c.name || '') + ' ' + (c.ownerName || '')).toLowerCase(); }).value();
    }

    function getBlacksmithsForEmployer(employer) {
        if (!employer) return [];
        return _.filter(getEmployerState(employer.id).craftsmen, function (c) { return c && c.type === 'blacksmith'; });
    }

    function getCraftsman(employerState, craftsmanId) {
        return _.find(employerState.craftsmen, function (c) { return c.id === craftsmanId; }) || null;
    }
    function getCraftsmanGlobal(craftsmanId) {
        return _.find(getAllBlacksmiths(), function (c) { return c && c.id === craftsmanId; }) || null;
    }
    function getQuote(employerState, quoteId) {
        return _.find(employerState.quotes, function (q) { return q.id === quoteId; }) || null;
    }
    function getJob(employerState, jobId) {
        return _.find(employerState.jobs, function (j) { return j.id === jobId; }) || null;
    }
    function getInspection(employerState, inspectionId) {
        return _.find(employerState.inspections, function (i) { return i.id === inspectionId; }) || null;
    }
    function removeQuote(employerState, quoteId) {
        employerState.quotes = _.filter(employerState.quotes, function (q) { return q.id !== quoteId; });
    }

    function smithTierLabel(tier) {
        switch (tier) {
            case 4: return 'Any sort of armor';
            case 3: return 'Ring / scale / studded / splint / chain';
            case 2: return 'Ring / scale / studded / splint';
            default: return 'Ring / scale / studded';
        }
    }

    function determineSmithSkill(race) {
        var raceData = CONFIG.race[race] || CONFIG.race.human;
        var raw = rollInteger(1, 100);
        var adjusted = Math.min(100, raw + raceData.skillBonus);
        var tier = 1;
        if (adjusted >= 81) tier = 4;
        else if (adjusted >= 66) tier = 3;
        else if (adjusted >= 41) tier = 2;
        return { rawRoll: raw, adjustedRoll: adjusted, tier: tier, label: smithTierLabel(tier) };
    }

    function raceEfficiency(race) {
        return (CONFIG.race[race] || CONFIG.race.human).efficiency;
    }

    function humanizeRole(role) {
        switch (role) {
            case 'leatherworker': return 'Leatherworker';
            case 'tailor': return 'Tailor';
            case 'woodworker': return 'Woodworker';
            default: return role;
        }
    }

    function specialistSupportCostCp(role, days) {
        var gpPerMonth = CONFIG.helperMonthlyGp[role] || 0;
        return Math.round((gpPerMonth / 30) * days * 100);
    }

    function arraySupportCostCp(support, days) {
        return _.reduce(support || [], function (sum, role) { return sum + specialistSupportCostCp(role, days); }, 0);
    }

    function canOfferProduct(craft, key) {
        var product = CONFIG.products[key];
        if (!product || !craft) return false;
        if (!product.isArmor) return true;
        return craft.skillTier >= product.skillMin;
    }

    function canRepairProduct(craft, key) {
        var product = CONFIG.products[key];
        if (!product || !craft || !product.isRepairableArmor) return false;
        return craft.skillTier >= product.skillMin;
    }

    function productActualDays(product, quantity, efficiency) {
        quantity = Math.max(1, parseInt(quantity || 1, 10));
        if (product.quantityBase && product.quantityBase > 1) {
            return Math.ceil(((quantity / product.quantityBase) * product.baseDays) / efficiency);
        }
        return Math.ceil((product.baseDays * quantity) / efficiency);
    }

    function productHumanDays(product, quantity) {
        quantity = Math.max(1, parseInt(quantity || 1, 10));
        if (product.quantityBase && product.quantityBase > 1) {
            return Math.ceil((quantity / product.quantityBase) * product.baseDays);
        }
        return Math.ceil(product.baseDays * quantity);
    }

    function repairActualDays(product, efficiency) {
        return Math.max(1, Math.ceil(Math.max(1, Math.ceil(product.baseDays / 4)) / Math.max(1, efficiency || 1)));
    }

    function smithQueryLabel(c) {
        var tier = smithTierLabel(c.skillTier).replace(/,/g, ';').replace(/\|/g, '/').replace(/\}/g, ')');
        var owner = c && c.ownerName ? (' - ' + String(c.ownerName).replace(/,/g, ';').replace(/\|/g, '/').replace(/\}/g, ')')) : '';
        return String(c.name || 'Blacksmith') + ' (' + tier + ')' + owner;
    }

    function productQueryLabel(product) {
        return String(product.label || 'Item').replace(/,/g, ';').replace(/\|/g, '/').replace(/\}/g, ')');
    }

    function buildProductQueryForSmith(craft, label) {
        var keys = _.filter(_.keys(CONFIG.products), function (key) { return canOfferProduct(craft, key); });
        if (!keys.length) return '?{' + label + '|}';
        return '?{' + label + '|' + _.map(keys, function (key) {
            return productQueryLabel(CONFIG.products[key]) + ',' + key;
        }).join('|') + '}';
    }

    function buildRepairProductQueryForSmith(craft, label) {
        var keys = _.filter(_.keys(CONFIG.products), function (key) { return canRepairProduct(craft, key); });
        if (!keys.length) return '?{' + label + '|}';
        return '?{' + label + '|' + _.map(keys, function (key) {
            return productQueryLabel(CONFIG.products[key]) + ',' + key;
        }).join('|') + '}';
    }

    function skillAvailabilityHtml(craft) {
        var available = [];
        var unavailable = [];
        _.each(_.keys(CONFIG.products), function (key) {
            var product = CONFIG.products[key];
            if (canOfferProduct(craft, key)) available.push(product.label);
            else unavailable.push(product.label);
        });
        available.sort();
        unavailable.sort();
        return sectionHeader('Item availability') +
            '<div style="margin:4px 0 6px 0;color:#7CFC7C;"><b>Can make:</b> ' + esc(available.join(', ')) + '</div>' +
            '<div style="margin:4px 0 8px 0;color:#ff8a8a;"><b>Cannot make:</b> ' + (unavailable.length ? esc(unavailable.join(', ')) : 'None') + '</div>';
    }

    function createQuote(employer, craft, options) {
        var employerState = getEmployerState(employer.id);
        var reactionBonus = Math.round(toNumber(getAttr(employer.id, CONFIG.reactionAttr, 0), 0));
        var days = Math.max(1, Math.ceil(options.days));
        var basePriceCp = gpToCp(options.basePriceGp || 0);
        var randomVariancePct = options.skipRandomVariance ? 0 : rollInteger(-15, 50);
        var randomAdjustedCp = Math.max(0, Math.round(basePriceCp * (1 + (randomVariancePct / 100))));
        var helperCostCp = arraySupportCostCp(options.support || [], days);
        var leatherCostCp = options.leather ? Math.round(basePriceCp * 0.10) : 0;
        var fixedCostCp = helperCostCp + leatherCostCp + gpToCp(options.extraCostGp || 0);
        var reactionAdjustedCp = Math.max(0, Math.round(randomAdjustedCp * (1 - (reactionBonus / 100))));
        var floorNegotiableCp = Math.round(basePriceCp * 0.30);
        var offeredNegotiableCp = Math.max(floorNegotiableCp, reactionAdjustedCp);
        var totalQuoteCp = fixedCostCp + offeredNegotiableCp;
        var depositCp = Math.ceil(totalQuoteCp / 2);
        var balanceCp = totalQuoteCp - depositCp;
        var quote = {
            id: generateQuoteId(),
            craftsmanId: craft.id,
            craftsmanName: craft.name,
            craftsmanTown: '',
            employerId: employer.id,
            employerName: employer.get('name') || 'Employer',
            description: options.description,
            quantity: options.quantity || 1,
            category: options.category || 'other',
            support: options.support || [],
            leather: !!options.leather,
            actualDays: days,
            humanDays: options.humanDays || days,
            basePriceCp: basePriceCp,
            randomVariancePct: randomVariancePct,
            randomAdjustedCp: randomAdjustedCp,
            floorNegotiableCp: floorNegotiableCp,
            fixedCostCp: fixedCostCp,
            helperCostCp: helperCostCp,
            leatherCostCp: leatherCostCp,
            extraCostCp: gpToCp(options.extraCostGp || 0),
            reactionBonus: reactionBonus,
            reactionAdjustedCp: reactionAdjustedCp,
            currentNegotiableCp: offeredNegotiableCp,
            currentQuoteCp: totalQuoteCp,
            depositCp: depositCp,
            balanceDueCp: balanceCp,
            haggleAttempts: 0,
            haggleFailures: 0,
            haggleDone: false,
            quoteType: options.quoteType || 'standard',
            productKey: options.productKey || '',
            notes: options.notes || ''
        };
        employerState.quotes.push(quote);
        return quote;
    }

    function quoteSummaryHtml(quote, showButtons) {
        var haggleTarget = Math.max(0, Math.min(95, 10 + toNumber(quote.reactionBonus, 0)));
        var body = '';
        body += line('Blacksmith', esc(quote.craftsmanName));
        body += line('Job', esc(quote.description));
        body += line('Actual days', esc(quote.actualDays));
        body += line('Starting table price', esc(formatCp(quote.basePriceCp)));
        if (quote.quoteType === 'repair') {
            body += line('Repair roll', esc((quote.repairPct || 0) + '% of listed price'));
            body += line('Inspection work', esc('1 day already spent'));
        } else {
            body += line('Random variance', esc((quote.randomVariancePct >= 0 ? '+' : '') + quote.randomVariancePct + '%'));
            body += line('After random variance', esc(formatCp(quote.randomAdjustedCp)));
        }
        body += line('Reaction bonus', esc((quote.reactionBonus >= 0 ? '+' : '') + quote.reactionBonus + '%'));
        body += line('After reaction adjustment', esc(formatCp(quote.currentNegotiableCp)));
        body += line('Haggle target', esc(haggleTarget + '%'));
        if (quote.support.length) body += line('Support trades', esc(_.map(quote.support, humanizeRole).join(', ')));
        if (quote.helperCostCp) body += line('Support trade time cost', esc(formatCp(quote.helperCostCp)));
        if (quote.leatherCostCp) body += line('Leather materials', esc(formatCp(quote.leatherCostCp)));
        if (quote.extraCostCp) body += line('Extra fixed cost', esc(formatCp(quote.extraCostCp)));
        if (quote.fixedCostCp) body += line('Material/helper costs', esc(formatCp(quote.fixedCostCp)));
        if (quote.notes) body += '<div style="margin-top:6px;color:' + CONFIG.colors.subtext + ';">' + esc(quote.notes) + '</div>';
        body += '<div style="margin-top:8px;padding-top:8px;border-top:1px solid #2e2e2e;">' +
            line('Final quote', '<span style="color:#ffd25e;font-weight:bold;">' + esc(formatCp(quote.currentQuoteCp)) + '</span>') +
            line('Half down now', esc(formatCp(quote.depositCp))) +
            line('Balance on pickup', esc(formatCp(quote.balanceDueCp))) + '</div>';
        if (showButtons) {
            body += '<div style="margin-top:8px;">' +
                btn('Place Order', '!smith-accept-quote --pc|' + quote.employerId + ' --quote|' + quote.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110) +
                (!quote.haggleDone ? btn('Haggle Price', '!smith-counteroffer --pc|' + quote.employerId + ' --quote|' + quote.id, CONFIG.colors.warnBg, CONFIG.colors.warnBorder, 110) : '') +
                btn('Deny Quote', '!smith-deny-quote --pc|' + quote.employerId + ' --quote|' + quote.id, CONFIG.colors.dangerBg, CONFIG.colors.dangerBorder, 110) +
                btn('Main Menu', '!smith-menu --pc|' + quote.employerId, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110) +
                '</div>';
        }
        return body;
    }

    function randomHaggleFailComment() {
        return CONFIG.haggleFailComments[rollInteger(1, CONFIG.haggleFailComments.length) - 1];
    }

    function menuBody(msg, employer) {
        var employerArg = ' --pc|@{selected|character_id}';
        var employerState = employer ? getEmployerState(employer.id) : null;
        var hasQuotes = employer && employerState && employerState.quotes.length;
        var infoLine = employer ? '<div style="margin-bottom:8px;color:' + CONFIG.colors.subtext + ';">Use the selected PC token as the buyer for this menu.</div>' : '<div style="margin-bottom:8px;color:' + CONFIG.colors.subtext + ';">Select the buyer PC token first.</div>';
        return infoLine +
            '<div style="margin-bottom:8px;">' +
            btn('Forge Armor', employer ? '!smith-quote-start' + employerArg : '!smith-menu', CONFIG.colors.primaryBg, CONFIG.colors.primaryBorder, 125) +
            btn('Armor Repairs', employer ? '!smith-repair-start' + employerArg : '!smith-menu', CONFIG.colors.warnBg, CONFIG.colors.warnBorder, 125) +
            '</div>' +
            '<div style="margin-bottom:8px;">' +
            btn('Price List', employer ? '!smith-price-list' + employerArg : '!smith-price-list', CONFIG.colors.primaryBg, CONFIG.colors.primaryBorder, 125) +
            btn('Work Progress', employer ? '!smith-report' + employerArg : '!smith-menu', CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 125) +
            '</div>' +
            '<div style="margin-bottom:8px;">' +
            btn('Apply Days', employer ? '!smith-apply-days' + employerArg + ' --days|?{Days passed|1}' : '!smith-menu', CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 125) +
            (hasQuotes ? btn('View Quotes', '!smith-quotes' + employerArg, CONFIG.colors.warnBg, CONFIG.colors.warnBorder, 125) : '') +
            '</div>' +
            (employer ? '<div style="margin-bottom:8px;">' +
                btn('Add Blacksmith', '!smith-add-blacksmith' + employerArg + ' --name|?{Blacksmith Name|' + randomName() + '} --race|?{Race|Human,human|Dwarf,dwarf|Gnome,gnome}', CONFIG.colors.primaryBg, CONFIG.colors.primaryBorder, 125) +
                '</div><div style="margin-bottom:8px;">' +
                btn('Remove Blacksmith', '!smith-remove-blacksmith-menu' + employerArg, CONFIG.colors.dangerBg, CONFIG.colors.dangerBorder, 125) +
                btn('Remove All Smiths', '!smith-remove-all' + employerArg + ' --confirm|?{Remove all blacksmiths and open quotes/jobs for this PC?|No,no|Yes,yes}', CONFIG.colors.dangerBg, CONFIG.colors.dangerBorder, 125) +
                '</div>' : '') +
            ((employer && playerIsGM(msg.playerid)) ? '<div style="margin-bottom:8px;">' +
                btnReset('Reset Campaign', '!smith-reset --confirm|?{Reset all Blacksmith data for the whole campaign?|No,no|Yes,yes}', 125) + '</div>' : '') +
            '<div style="color:' + CONFIG.colors.subtext + ';font-size:12px;">Armor repairs use a 1-day inspection. The repair quote is rolled from 10% to 90% of listed price, and there is a flat ' + CONFIG.repairNotRepairablePct + '% chance the item is beyond repair.</div>';
    }

    function handleMenu(msg, args) {
        send(box('Blacksmith Menu', menuBody(msg, resolveEmployer(msg, args))));
    }

    function handleAddBlacksmith(msg, args) {
        var employer = requireEmployer(msg, args);
        var employerState, race, skill, craftsman;
        if (!employer) return;
        employerState = getEmployerState(employer.id);
        race = String(args.race || 'human').toLowerCase();
        skill = determineSmithSkill(race);
        craftsman = {
            id: generateCraftsmanId(),
            type: 'blacksmith',
            name: args.name || randomName(),
            town: '',
            race: race,
            efficiency: raceEfficiency(race),
            skillRoll: skill.rawRoll,
            skillAdjusted: skill.adjustedRoll,
            skillTier: skill.tier,
            skillLabel: skill.label,
            setupCostCp: 0
        };
        employerState.craftsmen.push(craftsman);
        send(box('Blacksmith Added',
            line('Employer', esc(employer.get('name'))) +
            line('Name', esc(craftsman.name)) +
            line('Race', esc(craftsman.race)) +
            line('Raw skill roll', esc(craftsman.skillRoll)) +
            line('Adjusted skill roll', esc(craftsman.skillAdjusted)) +
            line('Armor skill', esc(craftsman.skillLabel)) +
            line('Efficiency', esc(craftsman.efficiency + 'x')) +
            skillAvailabilityHtml(craftsman) +
            btn('Forge Armor', '!smith-quote-menu --pc|' + employer.id + ' --smith|' + craftsman.id, CONFIG.colors.primaryBg, CONFIG.colors.primaryBorder, 110) +
            btn('Repair Armor', '!smith-repair-menu --pc|' + employer.id + ' --smith|' + craftsman.id, CONFIG.colors.warnBg, CONFIG.colors.warnBorder, 110) +
            btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)
        ));
    }

    function buildSmithButtons(employer, commandPrefix) {
        var smiths = getAllBlacksmiths();
        if (!smiths.length) return '<div>No blacksmiths found.</div>';
        commandPrefix = commandPrefix || '!smith-quote-menu';
        return _.map(smiths, function (smith) {
            return btn(smithQueryLabel(smith), commandPrefix + ' --pc|' + employer.id + ' --smith|' + smith.id, CONFIG.colors.primaryBg, CONFIG.colors.primaryBorder, 190);
        }).join('');
    }

    function handleQuoteStart(msg, args) {
        var employer = requireEmployer(msg, args);
        var smiths;
        if (!employer) return;
        smiths = getAllBlacksmiths();
        if (!smiths.length) {
            send(box('Forge Armor', '<div>No blacksmiths are recorded in the campaign yet.</div>' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
            return;
        }
        send(box('Forge Armor',
            '<div>Choose an available blacksmith. The selected token is the buyer.</div>' +
            buildSmithButtons(employer, '!smith-quote-menu') +
            '<div style="margin-top:8px;">' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110) + '</div>'
        ));
    }

    function handleRepairStart(msg, args) {
        var employer = requireEmployer(msg, args);
        var smiths;
        if (!employer) return;
        smiths = getAllBlacksmiths();
        if (!smiths.length) {
            send(box('Armor Repairs', '<div>No blacksmiths are recorded in the campaign yet.</div>' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
            return;
        }
        send(box('Armor Repairs',
            '<div>Choose a blacksmith to inspect the damaged armor. The selected token is the buyer.</div>' +
            buildSmithButtons(employer, '!smith-repair-menu') +
            '<div style="margin-top:8px;">' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110) + '</div>'
        ));
    }

    function forgeListNote(product) {
        if (product.key === 'paddedarmor') return 'Not forged here. See Tailor.';
        if (product.key === 'leatherarmor') return 'Not forged here. See Leatherworker.';
        if (product.key === 'studdedleather') return 'Mostly leather work. See Leatherworker and Tailor.';
        return product.notes || '';
    }

    function priceListRepairHtml(product) {
        var minCp = percentOfCp(gpToCp(product.basePriceGp || 0), CONFIG.repairCostMinPct);
        var maxCp = percentOfCp(gpToCp(product.basePriceGp || 0), CONFIG.repairCostMaxPct);
        return 'Repair quote after 1-day inspection: ' + esc(formatCp(minCp)) + ' to ' + esc(formatCp(maxCp)) + '.';
    }

    function handlePriceList(msg, args) {
        var employer = resolveEmployer(msg, args);
        var forgedItems = _.map(_.keys(CONFIG.products), function (key) {
            var product = CONFIG.products[key], note = forgeListNote(product), extra = '';
            if (product.isRepairableArmor) extra = ' - ' + priceListRepairHtml(product);
            return '<li><b>' + esc(product.label) + '</b> - forge/list price ' + esc(formatCp(gpToCp(product.basePriceGp || 0))) +
                ' - ' + esc(product.baseDays) + ' day(s)' + (note ? ' - ' + esc(note) : '') + extra + '</li>';
        }).join('');
        send(box('Blacksmith Price List',
            '<div style="color:' + CONFIG.colors.subtext + ';margin-bottom:8px;">Starting forge prices come from the equipment table and may vary by random variance, reaction bonus, helper costs, materials, and haggling. Armor repair inspection takes 1 day before a quote is generated.</div>' +
            '<div style="color:' + CONFIG.colors.subtext + ';margin-bottom:8px;">Repair quotes are rolled separately from 10% to 90% of listed price after inspection. If an item is not forged here, check the referral note in its entry.</div>' +
            '<ul style="margin:4px 0 0 18px;">' + forgedItems + '</ul>' +
            btn('Main Menu', employer ? '!smith-menu --pc|' + employer.id : '!smith-menu', CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)
        ));
    }

    function handleQuoteMenu(msg, args) {
        var employer = requireEmployer(msg, args);
        var employerState, craft, productQuery;
        if (!employer) return;
        employerState = getEmployerState(employer.id);
        craft = getCraftsmanGlobal(args.smith) || getCraftsman(employerState, args.smith);
        if (!craft || craft.type !== 'blacksmith') {
            send(box('Forge Armor', '<div>Pick an existing blacksmith first.</div>' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
            return;
        }
        productQuery = buildProductQueryForSmith(craft, 'Item');
        send(box('Forge Armor',
            line('Blacksmith', esc(craft.name)) +
            line('Armor skill', esc(craft.skillLabel + ' (' + craft.skillAdjusted + ')')) +
            skillAvailabilityHtml(craft) +
            '<div style="margin:8px 0;color:' + CONFIG.colors.subtext + ';">Select an item from the drop-down list. Starting prices come from the equipment table; final quotes vary.</div>' +
            btn('Quote Item', '!smith-quote-standard --pc|' + employer.id + ' --smith|' + craft.id + ' --job|' + productQuery + ' --qty|?{Quantity|1}', CONFIG.colors.primaryBg, CONFIG.colors.primaryBorder, 120) +
            btn('Armor Repairs', '!smith-repair-menu --pc|' + employer.id + ' --smith|' + craft.id, CONFIG.colors.warnBg, CONFIG.colors.warnBorder, 120) +
            btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)
        ));
    }

    function handleRepairMenu(msg, args) {
        var employer = requireEmployer(msg, args);
        var employerState, craft, repairQuery;
        if (!employer) return;
        employerState = getEmployerState(employer.id);
        craft = getCraftsmanGlobal(args.smith) || getCraftsman(employerState, args.smith);
        if (!craft || craft.type !== 'blacksmith') {
            send(box('Armor Repairs', '<div>Pick an existing blacksmith first.</div>' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
            return;
        }
        repairQuery = buildRepairProductQueryForSmith(craft, 'Armor to inspect');
        send(box('Armor Repairs',
            line('Blacksmith', esc(craft.name)) +
            line('Armor skill', esc(craft.skillLabel + ' (' + craft.skillAdjusted + ')')) +
            '<div style="margin:8px 0;color:' + CONFIG.colors.subtext + ';">Inspection takes 1 day. After that the smith decides whether the armor is repairable, then rolls a price from ' + CONFIG.repairCostMinPct + '% to ' + CONFIG.repairCostMaxPct + '% of listed PHB price.</div>' +
            btn('Start Inspection', '!smith-repair-inspect --pc|' + employer.id + ' --smith|' + craft.id + ' --job|' + repairQuery, CONFIG.colors.warnBg, CONFIG.colors.warnBorder, 125) +
            btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)
        ));
    }

    function handleQuoteStandard(msg, args) {
        var employer = requireEmployer(msg, args);
        var employerState, craft, product, qty, days, humanDays, quote;
        if (!employer) return;
        employerState = getEmployerState(employer.id);
        craft = getCraftsmanGlobal(args.smith) || getCraftsman(employerState, args.smith);
        product = CONFIG.products[String(args.job || '')];
        if (!craft || craft.type !== 'blacksmith') {
            send(box('Blacksmith Quote', '<div>Pick an existing blacksmith first.</div>' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
            return;
        }
        if (!product || !canOfferProduct(craft, String(args.job || ''))) {
            send(box('Blacksmith Quote',
                line('Employer', esc(employer.get('name'))) + line('Blacksmith', esc(craft.name)) +
                '<div>That item cannot be offered by this blacksmith at the current skill level.</div>' +
                btn('Quote Menu', '!smith-quote-menu --pc|' + employer.id + ' --smith|' + craft.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)
            ));
            return;
        }
        qty = Math.max(1, parseInt(args.qty || 1, 10));
        days = productActualDays(product, qty, craft.efficiency);
        humanDays = productHumanDays(product, qty);
        quote = createQuote(employer, craft, {
            description: product.label + (qty > 1 ? ' x' + qty : ''),
            quantity: qty,
            category: product.category,
            support: product.support,
            leather: product.leather,
            days: days,
            humanDays: humanDays,
            basePriceGp: (product.basePriceGp || 0) * qty,
            extraCostGp: 0,
            productKey: product.key,
            notes: product.notes,
            quoteType: 'standard'
        });
        send(box('Blacksmith Quote', quoteSummaryHtml(quote, true)));
    }

    function handleRepairInspect(msg, args) {
        var employer = requireEmployer(msg, args);
        var employerState, craft, product, inspection;
        if (!employer) return;
        employerState = getEmployerState(employer.id);
        craft = getCraftsmanGlobal(args.smith) || getCraftsman(employerState, args.smith);
        product = CONFIG.products[String(args.job || '')];
        if (!craft || craft.type !== 'blacksmith') {
            send(box('Armor Repairs', '<div>Pick an existing blacksmith first.</div>' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
            return;
        }
        if (!product || !canRepairProduct(craft, String(args.job || ''))) {
            send(box('Armor Repairs',
                line('Employer', esc(employer.get('name'))) + line('Blacksmith', esc(craft.name)) +
                '<div>That armor cannot be inspected by this blacksmith at the current skill level.</div>' +
                btn('Repair Menu', '!smith-repair-menu --pc|' + employer.id + ' --smith|' + craft.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)
            ));
            return;
        }
        inspection = {
            id: generateInspectionId(),
            craftsmanId: craft.id,
            craftsmanName: craft.name,
            employerId: employer.id,
            description: 'Repair inspection: ' + product.label,
            productKey: product.key,
            elapsedDays: 0,
            actualDays: 1,
            status: 'active',
            notes: 'Inspection underway.'
        };
        employerState.inspections.push(inspection);
        send(box('Inspection Started',
            line('Employer', esc(employer.get('name'))) +
            line('Blacksmith', esc(craft.name)) +
            line('Armor', esc(product.label)) +
            line('Inspection time', '1 day') +
            '<div>No quote is available until the inspection day passes.</div>' +
            btn('Apply Days', '!smith-apply-days --pc|' + employer.id + ' --days|1', CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110) +
            btn('Work Progress', '!smith-report --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)
        ));
    }

    function generateRepairQuoteFromInspection(employer, inspection) {
        var employerState = getEmployerState(employer.id);
        var craft = getCraftsmanGlobal(inspection.craftsmanId) || getCraftsman(employerState, inspection.craftsmanId);
        var product = CONFIG.products[String(inspection.productKey || '')];
        var repairableRoll = rollInteger(1, 100);
        var repairPct = rollInteger(CONFIG.repairCostMinPct, CONFIG.repairCostMaxPct);
        var quote, actualRepairDays, notes;
        if (!craft || !product) return null;
        if (repairableRoll <= CONFIG.repairNotRepairablePct) {
            inspection.status = 'complete';
            inspection.notes = 'Not repairable.';
            inspection.result = 'not-repairable';
            inspection.repairableRoll = repairableRoll;
            return {
                type: 'not-repairable',
                html: box('Repair Inspection Result',
                    line('Blacksmith', esc(craft.name)) +
                    line('Armor', esc(product.label)) +
                    line('Inspection roll', esc(repairableRoll)) +
                    line('Repairable threshold', esc(CONFIG.repairNotRepairablePct + '% not repairable')) +
                    '<div>The smith judges the item beyond repair after one day of inspection work.</div>' +
                    btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)
                )
            };
        }
        actualRepairDays = repairActualDays(product, craft.efficiency);
        notes = 'Repair quote generated after 1 inspection day. Repair time is estimated as roughly one-quarter of full production time.';
        quote = createQuote(employer, craft, {
            description: 'Repair ' + product.label,
            quantity: 1,
            category: product.category,
            support: product.support,
            leather: product.leather,
            days: actualRepairDays,
            humanDays: actualRepairDays,
            basePriceGp: (product.basePriceGp || 0) * (repairPct / 100),
            extraCostGp: 0,
            productKey: product.key,
            notes: notes,
            quoteType: 'repair',
            skipRandomVariance: true
        });
        quote.repairPct = repairPct;
        quote.repairableRoll = repairableRoll;
        inspection.status = 'complete';
        inspection.result = 'quoted';
        inspection.generatedQuoteId = quote.id;
        inspection.repairableRoll = repairableRoll;
        return { type: 'quote', quote: quote, html: box('Repair Inspection Result', quoteSummaryHtml(quote, true)) };
    }

    function handleCounteroffer(msg, args) {
        var employer = requireEmployer(msg, args);
        var employerState, quote, roll, success, haggleTarget, newNegotiableCp, resultText;
        if (!employer) return;
        employerState = getEmployerState(employer.id);
        quote = getQuote(employerState, args.quote);
        if (!quote) {
            send(box('Haggle Price', '<div>Quote not found.</div>' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
            return;
        }
        if (quote.haggleDone) {
            send(box('Haggle Price', line('Job', esc(quote.description)) + '<div>Haggling is already finished for this quote.</div>' + btn('View Quotes', '!smith-quotes --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
            return;
        }
        roll = rollInteger(1, 100);
        haggleTarget = Math.max(0, Math.min(95, 10 + toNumber(quote.reactionBonus, 0)));
        success = roll <= haggleTarget;
        quote.haggleAttempts = (quote.haggleAttempts || 0) + 1;
        if (success) {
            newNegotiableCp = Math.max(quote.floorNegotiableCp, Math.round(quote.currentNegotiableCp * 0.95));
            quote.currentNegotiableCp = newNegotiableCp;
            quote.currentQuoteCp = quote.fixedCostCp + newNegotiableCp;
            quote.depositCp = Math.ceil(quote.currentQuoteCp / 2);
            quote.balanceDueCp = quote.currentQuoteCp - quote.depositCp;
            quote.haggleDone = true;
            resultText = '<span style="color:#8fe38f;font-weight:bold;">Success - smith reduces the quoted smith charge by 5%.</span>';
        } else {
            quote.haggleFailures = (quote.haggleFailures || 0) + 1;
            quote.lastHaggleComment = randomHaggleFailComment();
            if (quote.haggleFailures >= 2) {
                quote.currentNegotiableCp = Math.round(quote.currentNegotiableCp * 1.10);
                quote.currentQuoteCp = quote.fixedCostCp + quote.currentNegotiableCp;
                quote.depositCp = Math.ceil(quote.currentQuoteCp / 2);
                quote.balanceDueCp = quote.currentQuoteCp - quote.depositCp;
                quote.haggleDone = true;
                resultText = '<span style="color:#ff7b7b;font-weight:bold;">Second failure - smith raises the quoted smith charge by 10% and ends haggling.</span>';
            } else {
                resultText = '<span style="color:#ffb36b;font-weight:bold;">Failure - no change yet. One more haggle attempt remains.</span>';
            }
        }
        send(box('Haggle Result',
            line('Job', esc(quote.description)) +
            line('Haggle target', esc(haggleTarget + '%')) +
            line('Die roll', esc(roll)) +
            line('Result', resultText) +
            ((!success && quote.lastHaggleComment) ? line('Smith says', esc(quote.lastHaggleComment)) : '') +
            line('Current quote', '<span style="color:#ffd25e;font-weight:bold;">' + esc(formatCp(quote.currentQuoteCp)) + '</span>') +
            line('Half down now', esc(formatCp(quote.depositCp))) +
            line('Balance on pickup', esc(formatCp(quote.balanceDueCp))) +
            '<div style="margin-top:8px;">' +
                btn('Place Order', '!smith-accept-quote --pc|' + employer.id + ' --quote|' + quote.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110) +
                (!quote.haggleDone ? btn('Haggle Price', '!smith-counteroffer --pc|' + employer.id + ' --quote|' + quote.id, CONFIG.colors.warnBg, CONFIG.colors.warnBorder, 110) : '') +
                (!success ? btn('Deny Quote', '!smith-deny-quote --pc|' + employer.id + ' --quote|' + quote.id, CONFIG.colors.dangerBg, CONFIG.colors.dangerBorder, 110) : '') +
                btn('View Quotes', '!smith-quotes --pc|' + employer.id, CONFIG.colors.warnBg, CONFIG.colors.warnBorder, 110) +
                btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110) +
            '</div>'
        ));
    }

    function handleAcceptQuote(msg, args) {
        var employer = requireEmployer(msg, args);
        var employerState, quote, payResult, job, depositCp, balanceCp;
        if (!employer) return;
        employerState = getEmployerState(employer.id);
        quote = getQuote(employerState, args.quote);
        if (!quote) {
            send(box('Place Order', '<div>Quote not found.</div>' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
            return;
        }
        depositCp = quote.depositCp || Math.ceil(quote.currentQuoteCp / 2);
        balanceCp = typeof quote.balanceDueCp === 'number' ? quote.balanceDueCp : (quote.currentQuoteCp - depositCp);
        payResult = payCostOrReport(employer, depositCp);
        if (!payResult.ok) {
            send(box('Place Order',
                line('Employer', esc(employer.get('name'))) +
                line('Needed now', esc(formatCp(payResult.needed))) +
                line('On hand', esc(formatCp(payResult.onHand))) +
                '<div>Not enough coin on the PC sheet for the half-down payment.</div>' +
                btn('View Quotes', '!smith-quotes --pc|' + employer.id, CONFIG.colors.warnBg, CONFIG.colors.warnBorder, 110)
            ));
            return;
        }
        job = {
            id: generateJobId(),
            craftsmanId: quote.craftsmanId,
            craftsmanName: quote.craftsmanName,
            description: quote.description,
            actualDays: quote.actualDays,
            elapsedDays: 0,
            status: 'active',
            finalCostCp: quote.currentQuoteCp,
            upfrontPaidCp: depositCp,
            balanceDueCp: balanceCp,
            support: quote.support || [],
            notes: 'Order placed. Half down paid: ' + formatCp(depositCp) + '. Haggle attempts: ' + (quote.haggleAttempts || 0) + '.',
            quoteType: quote.quoteType || 'standard'
        };
        employerState.jobs.push(job);
        removeQuote(employerState, quote.id);
        send(box('Order Placed',
            line('Employer', esc(employer.get('name'))) +
            line('Job', esc(job.description)) +
            line('Blacksmith', esc(job.craftsmanName)) +
            line('Half down paid', esc(formatCp(job.upfrontPaidCp))) +
            line('Balance due on pickup', esc(formatCp(job.balanceDueCp))) +
            line('Time to complete', esc(job.actualDays + ' day(s)')) +
            btn('Work Progress', '!smith-report --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110) +
            btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)
        ));
    }

    function handleDenyQuote(msg, args) {
        var employer = requireEmployer(msg, args);
        var employerState, quote;
        if (!employer) return;
        employerState = getEmployerState(employer.id);
        quote = getQuote(employerState, args.quote);
        if (!quote) {
            send(box('Deny Quote', '<div>Quote not found.</div>' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
            return;
        }
        removeQuote(employerState, quote.id);
        send(box('Quote Denied',
            line('Employer', esc(employer.get('name'))) +
            line('Blacksmith', esc(quote.craftsmanName)) +
            line('Job', esc(quote.description)) +
            '<div>The quote was removed.</div>' +
            btn('View Quotes', '!smith-quotes --pc|' + employer.id, CONFIG.colors.warnBg, CONFIG.colors.warnBorder, 110) +
            btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)
        ));
    }

    function handleQuotes(msg, args) {
        var employer = requireEmployer(msg, args);
        var employerState, body;
        if (!employer) return;
        employerState = getEmployerState(employer.id);
        if (!employerState.quotes.length) {
            send(box('Open Quotes', line('Employer', esc(employer.get('name'))) + '<div>No open quotes.</div>' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
            return;
        }
        body = line('Employer', esc(employer.get('name'))) + _.map(employerState.quotes, function (q) {
            return '<div style="margin:10px 0;padding-top:8px;border-top:1px solid #2e2e2e;">' + quoteSummaryHtml(q, true) + '</div>';
        }).join('');
        send(box('Open Quotes', body));
    }

    function handleApplyDays(msg, args) {
        var employer = requireEmployer(msg, args);
        var employerState, days, changed, outputs = [];
        if (!employer) return;
        employerState = getEmployerState(employer.id);
        days = Math.max(0, parseInt(args.days || 0, 10));
        changed = 0;
        _.each(employerState.jobs, function (j) {
            if (j.status !== 'active') return;
            j.elapsedDays += days;
            if (j.elapsedDays >= j.actualDays) j.status = 'ready';
            changed += 1;
        });
        _.each(employerState.inspections, function (i) {
            if (i.status !== 'active') return;
            i.elapsedDays += days;
            if (i.elapsedDays >= i.actualDays) {
                var result = generateRepairQuoteFromInspection(employer, i);
                if (result && result.html) outputs.push(result.html);
            }
        });
        employerState.inspections = _.filter(employerState.inspections, function (i) { return i.status !== 'complete'; });
        send(box('Days Applied',
            line('Employer', esc(employer.get('name'))) +
            line('Days added', esc(days)) +
            line('Active jobs updated', esc(changed)) +
            btn('Work Progress', '!smith-report --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110) +
            btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)
        ));
        _.each(outputs, function (html) { send(html); });
    }

    function handleCompleteJob(msg, args) {
        var employer = requireEmployer(msg, args);
        var employerState, job, payResult, paidCp;
        if (!employer) return;
        employerState = getEmployerState(employer.id);
        job = getJob(employerState, args.job);
        if (!job) {
            send(box('Pick Up Job', '<div>Job not found.</div>' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
            return;
        }
        if (job.status === 'active') {
            send(box('Pick Up Job',
                line('Job', esc(job.description)) +
                '<div>The job is not ready yet.</div>' +
                btn('Work Progress', '!smith-report --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)
            ));
            return;
        }
        paidCp = Math.max(0, toNumber(job.balanceDueCp, 0));
        payResult = payCostOrReport(employer, paidCp);
        if (!payResult.ok) {
            send(box('Pick Up Job',
                line('Employer', esc(employer.get('name'))) +
                line('Balance needed', esc(formatCp(payResult.needed))) +
                line('On hand', esc(formatCp(payResult.onHand))) +
                '<div>Not enough coin on the PC sheet to pay the remaining balance.</div>' +
                btn('Work Progress', '!smith-report --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)
            ));
            return;
        }
        employerState.jobs = _.filter(employerState.jobs, function (j) { return j.id !== job.id; });
        send(box('Job Picked Up',
            line('Employer', esc(employer.get('name'))) +
            line('Job', esc(job.description)) +
            line('Blacksmith', esc(job.craftsmanName)) +
            line('Balance paid', esc(formatCp(paidCp))) +
            '<div>The completed job was removed from the active jobs list.</div>' +
            btn('Work Progress', '!smith-report --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110) +
            btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)
        ));
    }

    function handleReport(msg, args) {
        var employer = requireEmployer(msg, args);
        var employerState, craftsmenHtml, jobsHtml, quotesHtml, inspectionsHtml, body;
        if (!employer) return;
        employerState = getEmployerState(employer.id);
        if (employerState.craftsmen.length) {
            craftsmenHtml = '<ul style="margin:6px 0 6px 18px;">' + _.map(employerState.craftsmen, function (c) {
                return '<li><b>' + esc(c.name) + '</b> - ' + esc(c.race) + ' - ' + esc(c.skillLabel) + ' (' + esc(c.skillAdjusted) + ') - efficiency ' + esc(c.efficiency) + 'x</li>';
            }).join('') + '</ul>';
        } else craftsmenHtml = '<div>No blacksmiths recorded.</div>';

        inspectionsHtml = employerState.inspections.length ? '<ul style="margin:6px 0 6px 18px;">' + _.map(employerState.inspections, function (i) {
            var remaining = Math.max(0, i.actualDays - i.elapsedDays);
            return '<li><b>' + esc(i.description) + '</b> - ' + esc(i.craftsmanName) + ' - ' + esc(i.status) + ' - elapsed ' +
                esc(i.elapsedDays) + '/' + esc(i.actualDays) + ' days - remaining ' + esc(remaining) + ' day(s)</li>';
        }).join('') + '</ul>' : '<div>No active inspections.</div>';

        jobsHtml = employerState.jobs.length ? '<ul style="margin:6px 0 6px 18px;">' + _.map(employerState.jobs, function (j) {
            var remaining = Math.max(0, j.actualDays - j.elapsedDays);
            return '<li><b>' + esc(j.description) + '</b> - ' + esc(j.craftsmanName) + ' - ' + esc(j.status) +
                ' - elapsed ' + esc(j.elapsedDays) + '/' + esc(j.actualDays) + ' days - remaining ' + esc(remaining) + ' day(s)' +
                ' - total ' + esc(formatCp(j.finalCostCp)) +
                ' - down ' + esc(formatCp(j.upfrontPaidCp || 0)) +
                ' - balance ' + esc(formatCp(j.balanceDueCp || 0)) +
                (j.status !== 'active' ? ' ' + btn('Pick Up', '!smith-complete-job --pc|' + employer.id + ' --job|' + j.id, CONFIG.colors.warnBg, CONFIG.colors.warnBorder, 85) : '') +
                '</li>';
        }).join('') + '</ul>' : '<div>No active jobs.</div>';

        quotesHtml = employerState.quotes.length ? '<ul style="margin:6px 0 6px 18px;">' + _.map(employerState.quotes, function (q) {
            return '<li><b>' + esc(q.description) + '</b> - ' + esc(q.craftsmanName) + ' - ' + esc(formatCp(q.currentQuoteCp)) + '</li>';
        }).join('') + '</ul>' : '<div>No open quotes.</div>';

        body = line('Employer', esc(employer.get('name'))) +
            sectionHeader('Blacksmiths') + craftsmenHtml +
            sectionHeader('Open Inspections') + inspectionsHtml +
            sectionHeader('Open Quotes') + quotesHtml +
            sectionHeader('Active Jobs') + jobsHtml +
            '<div style="margin-top:8px;">' +
            btn('View Quotes', '!smith-quotes --pc|' + employer.id, CONFIG.colors.warnBg, CONFIG.colors.warnBorder, 110) +
            btn('Apply Days', '!smith-apply-days --pc|' + employer.id + ' --days|?{Days passed|1}', CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110) +
            btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110) +
            '</div>';
        send(box('Work Progress', body));
    }

    function handleRemoveBlacksmithMenu(msg, args) {
        var employer = requireEmployer(msg, args);
        var employerState, smiths;
        if (!employer) return;
        employerState = getEmployerState(employer.id);
        smiths = _.filter(employerState.craftsmen, function (c) { return c.type === 'blacksmith'; });
        if (!smiths.length) {
            send(box('Remove Blacksmith', '<div>No blacksmiths recorded for this employer.</div>' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
            return;
        }
        send(box('Remove Blacksmith',
            _.map(smiths, function (c) {
                return '<div style="margin-bottom:4px;">' + esc(c.name) + ' ' + btn('Remove', '!smith-remove-blacksmith --pc|' + employer.id + ' --smith|' + c.id, CONFIG.colors.dangerBg, CONFIG.colors.dangerBorder, 80) + '</div>';
            }).join('') + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)
        ));
    }

    function handleRemoveBlacksmith(msg, args) {
        var employer = requireEmployer(msg, args);
        var employerState, craft;
        if (!employer) return;
        employerState = getEmployerState(employer.id);
        craft = getCraftsman(employerState, args.smith);
        if (!craft) {
            send(box('Remove Blacksmith', '<div>Blacksmith not found.</div>' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
            return;
        }
        employerState.craftsmen = _.filter(employerState.craftsmen, function (c) { return c.id !== craft.id; });
        employerState.quotes = _.filter(employerState.quotes, function (q) { return q.craftsmanId !== craft.id; });
        employerState.jobs = _.filter(employerState.jobs, function (j) { return j.craftsmanId !== craft.id; });
        employerState.inspections = _.filter(employerState.inspections, function (i) { return i.craftsmanId !== craft.id; });
        send(box('Remove Blacksmith',
            line('Removed', esc(craft.name)) +
            '<div>Any open quotes, inspections, and jobs for that blacksmith were removed.</div>' +
            btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)
        ));
    }

    function handleRemoveAll(msg, args) {
        var employer = requireEmployer(msg, args);
        var employerState;
        if (!employer) return;
        if (String(args.confirm || 'no').toLowerCase() !== 'yes') {
            send(box('Remove All Smiths', '<div>No changes made.</div>' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
            return;
        }
        employerState = getEmployerState(employer.id);
        employerState.craftsmen = [];
        employerState.quotes = [];
        employerState.jobs = [];
        employerState.inspections = [];
        send(box('Remove All Smiths', '<div>All blacksmiths, quotes, inspections, and jobs were cleared for this PC.</div>' + btn('Main Menu', '!smith-menu --pc|' + employer.id, CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
    }

    function handleReset(msg, args) {
        if (!playerIsGM(msg.playerid)) {
            send(box('Blacksmith', '<div>GM only.</div>'));
            return;
        }
        if (String(args.confirm || 'no').toLowerCase() !== 'yes') {
            send(box('Reset Campaign', '<div>No changes made.</div>' + btn('Main Menu', '!smith-menu', CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
            return;
        }
        state[STATE] = { version: VERSION, nextCraftsmanId: 1, nextQuoteId: 1, nextJobId: 1, nextInspectionId: 1, employers: {} };
        send(box('Reset Campaign', '<div>All Blacksmith data, quotes, inspections, and jobs were cleared for the campaign.</div>' + btn('Main Menu', '!smith-menu', CONFIG.colors.reportBg, CONFIG.colors.reportBorder, 110)));
    }

    function handleHelp() {
        send(box('Blacksmith Help',
            '<div>Use the menu. Select the buyer PC token first.</div>' +
            '<ul><li>!smith-menu</li><li>!smith-report --pc|CHAR_ID</li><li>!smith-apply-days --pc|CHAR_ID --days|1</li></ul>'
        ));
    }

    function handleInput(msg) {
        var cmd, args, player;
        if (msg.type !== 'api' || !msg.content || msg.content.indexOf('!smith') !== 0) return;
        player = getObjById('player', msg.playerid);
        whisperTarget = player ? String(player.get('_displayname') || '') : String(msg.who || '').replace(/\s+\(GM\)$/, '');
        cmd = msg.content.split(/\s+/)[0];
        args = parseArgs(msg.content);
        switch (cmd) {
            case '!smith-menu': handleMenu(msg, args); break;
            case '!smith-help': handleHelp(); break;
            case '!smith-add-blacksmith': handleAddBlacksmith(msg, args); break;
            case '!smith-quote-start': handleQuoteStart(msg, args); break;
            case '!smith-price-list': handlePriceList(msg, args); break;
            case '!smith-quote-menu': handleQuoteMenu(msg, args); break;
            case '!smith-quote-standard': handleQuoteStandard(msg, args); break;
            case '!smith-repair-start': handleRepairStart(msg, args); break;
            case '!smith-repair-menu': handleRepairMenu(msg, args); break;
            case '!smith-repair-inspect': handleRepairInspect(msg, args); break;
            case '!smith-counteroffer': handleCounteroffer(msg, args); break;
            case '!smith-accept-quote': handleAcceptQuote(msg, args); break;
            case '!smith-deny-quote': handleDenyQuote(msg, args); break;
            case '!smith-remove-blacksmith-menu': handleRemoveBlacksmithMenu(msg, args); break;
            case '!smith-remove-blacksmith': handleRemoveBlacksmith(msg, args); break;
            case '!smith-remove-all': handleRemoveAll(msg, args); break;
            case '!smith-reset': handleReset(msg, args); break;
            case '!smith-quotes': handleQuotes(msg, args); break;
            case '!smith-apply-days': handleApplyDays(msg, args); break;
            case '!smith-report': handleReport(msg, args); break;
            case '!smith-complete-job': handleCompleteJob(msg, args); break;
            default: handleHelp(); break;
        }
    }

    return { CheckInstall: checkInstall, RegisterEventHandlers: registerEventHandlers };
}());

on('ready', function () {
    'use strict';
    StrongholdSmithy.CheckInstall();
    StrongholdSmithy.RegisterEventHandlers();
});
