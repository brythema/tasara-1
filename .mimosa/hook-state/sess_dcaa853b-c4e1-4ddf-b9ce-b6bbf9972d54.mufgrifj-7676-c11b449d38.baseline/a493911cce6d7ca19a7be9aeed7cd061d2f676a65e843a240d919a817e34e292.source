/* ============================================================
   TASARA — shared site script (v3 — streamlined)
   Auth, API, page transitions, reveal-on-scroll, mobile nav.
   ============================================================ */

// ── Page Transition (reduced theatrics) ─────────────────────
(function(){
  let isTransitioning = false;
  function triggerTransition(targetUrl) {
    if (isTransitioning || !targetUrl) return;
    isTransitioning = true;
    const content = document.querySelector('.page-content');
    if (content) {
      content.style.opacity = '0';
      content.style.transition = 'opacity 0.3s ease';
    }
    setTimeout(() => { window.location.href = targetUrl; }, 320);
  }
  document.addEventListener('click', function(e) {
    const link = e.target.closest('a[href]');
    if (!link) return;
    const href = link.getAttribute('href');
    if (href && !href.startsWith('http') && !href.startsWith('#') && !href.startsWith('mailto:') && !href.startsWith('tel:')) {
      e.preventDefault();
      triggerTransition(href);
    }
  }, true);
  window.tasaraNavigate = triggerTransition;
})();

// ── Helpers ──────────────────────────────────────────────────
const API = '/api';
function escapeHtml(str) {
  return String(str == null ? '' : str).replace(/[&<>"']/g, function(c) {
    return { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c];
  });
}
function getToken() { return localStorage.getItem('tasara_token'); }
function setToken(t) { localStorage.setItem('tasara_token', t); }
function clearAuth() { localStorage.removeItem('tasara_token'); localStorage.removeItem('tasara_user'); }
function authHeaders() {
  const t = getToken();
  return t ? { Authorization: 'Bearer ' + t } : {};
}
async function apiFetch(path, options = {}) {
  const res = await fetch(API + path, {
    headers: { 'Content-Type': 'application/json', ...authHeaders(), ...options.headers },
    ...options,
  });
  const data = await res.json().catch(() => ({}));
  if (!res.ok) throw new Error(data.error || 'HTTP ' + res.status);
  return data;
}
function requireAuth(redirectTo = 'login.html') {
  if (!getToken()) { window.location.href = redirectTo; return false; }
  return true;
}

// ── Init ─────────────────────────────────────────────────────
document.addEventListener('DOMContentLoaded', function () {
  var page = document.body.getAttribute('data-page');

  // Topbar scroll behavior
  (function () {
    var topbar = document.querySelector('.topbar');
    if (!topbar) return;
    function onScroll() { topbar.classList.toggle('scrolled', window.scrollY > 50); }
    onScroll();
    window.addEventListener('scroll', onScroll, { passive: true });
  })();

  // Mobile nav
  (function () {
    var burger = document.getElementById('burger');
    var menu = document.getElementById('mobileMenu');
    if (burger && menu) {
      burger.addEventListener('click', function () {
        var open = menu.classList.toggle('open');
        burger.setAttribute('aria-expanded', open);
        burger.textContent = open ? '✕' : '☰';
        if (open) menu.querySelector('a').focus();
      });
      // Close on link click
      menu.querySelectorAll('a').forEach(function(a) {
        a.addEventListener('click', function() {
          menu.classList.remove('open');
          burger.setAttribute('aria-expanded', 'false');
          burger.textContent = '☰';
        });
      });
      // Escape to close
      document.addEventListener('keydown', function(e) {
        if (e.key === 'Escape' && menu.classList.contains('open')) {
          menu.classList.remove('open');
          burger.setAttribute('aria-expanded', 'false');
          burger.textContent = '☰';
          burger.focus();
        }
      });
    }
  })();

  // Reveal on scroll
  (function () {
    var els = document.querySelectorAll('.reveal');
    if (!els.length) return;
    if (!('IntersectionObserver' in window)) {
      els.forEach(function(el) { el.classList.add('in-view'); });
      return;
    }
    var io = new IntersectionObserver(function(entries) {
      entries.forEach(function(entry) {
        if (entry.isIntersecting) {
          entry.target.classList.add('in-view');
          io.unobserve(entry.target);
        }
      });
    }, { threshold: 0.1, rootMargin: '0px 0px -30px 0px' });
    els.forEach(function(el) { io.observe(el); });
  })();

  // Counter animation
  (function () {
    var counters = document.querySelectorAll('.stat-num[data-count]');
    counters.forEach(function(el) {
      el.classList.add('reveal');
      var target = parseInt(el.dataset.count);
      var prefix = el.textContent.startsWith('$') ? '$' : '';
      var suffix = el.textContent.includes('+') ? '+' : '';
      function animate() {
        var current = 0;
        var step = Math.ceil(target / 50);
        var timer = setInterval(function() {
          current += step;
          if (current >= target) { current = target; clearInterval(timer); }
          el.textContent = prefix + current.toLocaleString() + suffix;
        }, 30);
      }
      var obs = new IntersectionObserver(function(entries) {
        if (entries[0].isIntersecting) { animate(); obs.unobserve(el); }
      }, { threshold: 0.5 });
      obs.observe(el);
    });
  })();

  // FAQ accordion
  (function () {
    document.querySelectorAll('.faq-question').forEach(function(btn) {
      btn.addEventListener('click', function() {
        var item = btn.closest('.faq-item');
        var faqList = btn.closest('.faq-list');
        if (!faqList) return;
        var isOpen = item.classList.contains('open');
        // Close all in same list
        faqList.querySelectorAll('.faq-item').forEach(function(i) {
          i.classList.remove('open');
          i.querySelector('.faq-question').setAttribute('aria-expanded', 'false');
        });
        if (!isOpen) {
          item.classList.add('open');
          btn.setAttribute('aria-expanded', 'true');
        }
      });
    });
  })();

  // Logout
  var logoutLink = document.getElementById('logoutLink');
  if (logoutLink) {
    logoutLink.addEventListener('click', function(e) {
      e.preventDefault();
      clearAuth();
      window.location.href = 'login.html';
    });
  }

  // ── Login ───────────────────────────────────────────────
  if (page === 'login') {
    var modeSwitch = document.getElementById('modeSwitch');
    var flipCard = document.getElementById('flipCard');
    var modeLabel = document.getElementById('modeLabel');
    var stageEyebrow = document.getElementById('stageEyebrow');
    var stageTitle = document.getElementById('stageTitle');
    var stageSub = document.getElementById('stageSub');

    function setAdminMode(on) {
      modeSwitch.classList.toggle('on', on);
      modeSwitch.setAttribute('aria-pressed', on);
      flipCard.classList.toggle('is-flipped', on);
      if (on) {
        modeLabel.textContent = 'Administrator sign-in active';
        stageEyebrow.textContent = 'Administrator access';
        stageTitle.textContent = 'Enter the admin console';
        stageSub.textContent = 'For authorized TASARA Administrators reviewing Merchant Steward applications and network activity.';
      } else {
        modeLabel.textContent = 'This is an administrator sign-in';
        stageEyebrow.textContent = 'Sign in';
        stageTitle.textContent = 'Welcome back to TASARA';
        stageSub.textContent = 'Sign in to reach your dashboard, track your Omni activity, and stay connected with the network.';
      }
    }
    if (modeSwitch) modeSwitch.addEventListener('click', function() { setAdminMode(!modeSwitch.classList.contains('on')); });
    if (modeSwitch) modeSwitch.addEventListener('keydown', function(e) { if (e.key === 'Enter' || e.key === ' ') { e.preventDefault(); modeSwitch.click(); } });
    var backToNormal = document.getElementById('backToNormal');
    if (backToNormal) backToNormal.addEventListener('click', function(e) { e.preventDefault(); setAdminMode(false); });

    var frontForm = document.getElementById('frontForm');
    if (frontForm) {
      frontForm.addEventListener('submit', async function(e) {
        e.preventDefault();
        var email = document.getElementById('fEmail').value.trim();
        var password = document.getElementById('fPass').value;
        var errorBox = document.getElementById('frontError');
        errorBox.classList.remove('show');
        try {
          var result = await apiFetch('/auth/login', { method: 'POST', body: JSON.stringify({ email, password }) });
          setToken(result.token);
          localStorage.setItem('tasara_user', JSON.stringify(result.user));
          var target = result.user.role === 'admin' ? 'dashboard-admin.html'
            : result.user.role === 'seller' ? 'dashboard-seller.html'
            : 'dashboard-buyer.html';
          window.location.href = target;
        } catch (err) {
          errorBox.textContent = err.message || 'Login failed. Check your credentials.';
          errorBox.classList.add('show');
        }
      });
    }

    var backForm = document.getElementById('backForm');
    if (backForm) {
      backForm.addEventListener('submit', async function(e) {
        e.preventDefault();
        var email = document.getElementById('aEmail').value.trim();
        var password = document.getElementById('aPass').value;
        var errorBox = document.getElementById('backError');
        errorBox.classList.remove('show');
        try {
          var result = await apiFetch('/auth/login', { method: 'POST', body: JSON.stringify({ email, password }) });
          if (result.user.role !== 'admin') throw new Error('Access denied — not an administrator account');
          setToken(result.token);
          localStorage.setItem('tasara_user', JSON.stringify(result.user));
          window.location.href = 'dashboard-admin.html';
        } catch (err) {
          errorBox.textContent = err.message || 'Invalid admin credentials.';
          errorBox.classList.add('show');
        }
      });
    }
  }

  // ── Signup ──────────────────────────────────────────────
  if (page === 'signup') {
    var tabBuyer = document.getElementById('tabBuyer');
    var tabSeller = document.getElementById('tabSeller');
    var buyerPanel = document.getElementById('buyerPanel');
    var sellerPanel = document.getElementById('sellerPanel');
    var stageTitle = document.getElementById('stageTitle');
    var stageSub = document.getElementById('stageSub');
    var loopCaption = document.getElementById('loopCaption');

    function showBuyer() {
      tabBuyer.classList.add('active'); tabBuyer.setAttribute('aria-selected', 'true');
      tabSeller.classList.remove('active'); tabSeller.setAttribute('aria-selected', 'false');
      buyerPanel.style.display = 'block'; sellerPanel.style.display = 'none';
      stageTitle.textContent = 'Join TASARA';
      stageSub.textContent = 'Create a free account to browse the Directory and submit requests to Merchant Stewards.';
      loopCaption.innerHTML = 'TASARA gives your work an identity within a trusted network — <b>seen, tracked, and valued over time.</b>';
    }
    function showSeller() {
      tabSeller.classList.add('active'); tabSeller.setAttribute('aria-selected', 'true');
      tabBuyer.classList.remove('active'); tabBuyer.setAttribute('aria-selected', 'false');
      sellerPanel.style.display = 'block'; buyerPanel.style.display = 'none';
      stageTitle.textContent = 'Become a Merchant Steward';
      stageSub.textContent = 'Register your business, choose your Stewardship tier, and submit for Administrator approval.';
      loopCaption.innerHTML = 'Every Merchant Steward is verified before going live — <b>your standing is built, not assumed.</b>';
    }
    if (tabBuyer) tabBuyer.addEventListener('click', showBuyer);
    if (tabSeller) tabSeller.addEventListener('click', showSeller);

    // Deep link support: signup.html?role=seller opens straight on the Seller tab
    var initialRole = new URLSearchParams(window.location.search).get('role');
    if (initialRole === 'seller') showSeller();

    var buyerForm = document.getElementById('buyerForm');
    if (buyerForm) {
      buyerForm.addEventListener('submit', async function(e) {
        e.preventDefault();
        var name = document.getElementById('bName').value.trim();
        var email = document.getElementById('bEmail').value.trim();
        var phone = document.getElementById('bPhone').value.trim();
        var address = document.getElementById('bAddress').value.trim();
        var password = document.getElementById('bPassword').value;
        var passwordConfirm = document.getElementById('bPasswordConfirm').value;
        var err = document.getElementById('buyerError');
        if (!name || !email || !phone || !address || !password || !passwordConfirm) { err.textContent = 'Please fill in every field.'; err.classList.add('show'); return; }
        if (password.length < 8) { err.textContent = 'Password must be at least 8 characters.'; err.classList.add('show'); return; }
        if (password !== passwordConfirm) { err.textContent = 'Passwords do not match.'; err.classList.add('show'); return; }
        err.classList.remove('show');
        try {
          var result = await apiFetch('/auth/register', {
            method: 'POST',
            body: JSON.stringify({ full_name: name, email, phone, password, address, role: 'buyer' })
          });
          setToken(result.token);
          localStorage.setItem('tasara_user', JSON.stringify(result.user));
          window.location.href = 'dashboard-buyer.html';
        } catch (apiErr) {
          err.textContent = apiErr.message || 'Registration failed. Try again.';
          err.classList.add('show');
        }
      });
    }

    var panels = document.querySelectorAll('.step-panel');
    var dots = document.querySelectorAll('.stepper .dot');
    var stepLabel = document.getElementById('stepLabel');
    var stepBack = document.getElementById('stepBack');
    var stepNext = document.getElementById('stepNext');
    var stepSubmit = document.getElementById('stepSubmit');
    var sellerError = document.getElementById('sellerError');
    var current = 1;
    var total = 3;
    var labels = { 1: 'Your details', 2: 'Choose your tier', 3: 'Verification' };

    function renderStep() {
      panels.forEach(function(p) { p.classList.toggle('active', Number(p.dataset.panel) === current); });
      dots.forEach(function(d) { d.classList.toggle('done', Number(d.dataset.step) <= current); });
      stepLabel.innerHTML = 'Step <b>' + current + ' of ' + total + '</b> — ' + labels[current];
      stepBack.style.display = current === 1 ? 'none' : 'inline-flex';
      stepNext.style.display = current === total ? 'none' : 'inline-flex';
      stepSubmit.style.display = current === total ? 'inline-flex' : 'none';
      if (sellerError) sellerError.classList.remove('show');
    }

    function validateStep() {
      var panel = document.querySelector('.step-panel[data-panel="' + current + '"]');
      if (!panel) return true;
      var inputs = panel.querySelectorAll('input[required]');
      for (var i = 0; i < inputs.length; i++) {
        var inp = inputs[i];
        if (inp.type === 'radio') {
          var group = panel.querySelectorAll('input[name="' + inp.name + '"]');
          var checked = false;
          for (var j = 0; j < group.length; j++) { if (group[j].checked) { checked = true; break; } }
          if (!checked) return false;
        } else if (!inp.value.trim()) {
          return false;
        }
      }
      return true;
    }

    if (stepNext) stepNext.addEventListener('click', function() {
      if (!validateStep()) { if (sellerError) { sellerError.textContent = 'Please complete this step.'; sellerError.classList.add('show'); } return; }
      current = Math.min(total, current + 1);
      renderStep();
    });
    if (stepBack) stepBack.addEventListener('click', function() { current = Math.max(1, current - 1); renderStep(); });

    var uploadBox = document.getElementById('uploadBox');
    var idUpload = document.getElementById('idUpload');
    var uploadText = document.getElementById('uploadText');
    var uploadFilename = document.getElementById('uploadFilename');
    if (idUpload) {
      idUpload.addEventListener('change', function() {
        if (idUpload.files.length) {
          uploadBox.classList.add('has-file');
          uploadText.textContent = 'File ready — tap to replace';
          if (uploadFilename) uploadFilename.textContent = idUpload.files[0].name;
        }
      });
    }

    // Bound to the form's submit event (not just a button click) so that
    // pressing Enter in any step-1/2 field is handled the same way instead
    // of falling through to the browser's native form submission.
    var sellerForm = document.getElementById('sellerForm');
    if (sellerForm) {
      sellerForm.addEventListener('submit', async function(e) {
        e.preventDefault();

        // Enter pressed before the final step: advance instead of submitting.
        if (current !== total) {
          if (!validateStep()) { if (sellerError) { sellerError.textContent = 'Please complete this step.'; sellerError.classList.add('show'); } return; }
          current = Math.min(total, current + 1);
          renderStep();
          return;
        }

        if (!validateStep()) { if (sellerError) { sellerError.textContent = 'Please complete all steps.'; sellerError.classList.add('show'); } return; }

        var name = document.getElementById('sName').value.trim();
        var email = document.getElementById('sEmail').value.trim();
        var phone = document.getElementById('sPhone').value.trim();
        var address = document.getElementById('sAddress').value.trim();
        var bizName = document.getElementById('sBizName').value.trim();
        var bizLoc = document.getElementById('sBizLocation').value.trim();
        var password = document.getElementById('sPassword').value;
        var passwordConfirm = document.getElementById('sPasswordConfirm').value;
        var tierValue = document.querySelector('input[name="tier"]:checked');
        var err = document.getElementById('sellerError');
        if (!tierValue) { err.textContent = 'Please select a tier.'; err.classList.add('show'); return; }
        if (password.length < 8) { err.textContent = 'Password must be at least 8 characters.'; err.classList.add('show'); return; }
        if (password !== passwordConfirm) { err.textContent = 'Passwords do not match.'; err.classList.add('show'); return; }

        var tier = tierValue.value;
        err.classList.remove('show');

        try {
          var regResult = await apiFetch('/auth/register', {
            method: 'POST',
            body: JSON.stringify({
              full_name: name, email, phone,
              password: password, address,
              role: 'seller',
              business_name: bizName, business_location: bizLoc, tier: parseInt(tier)
            })
          });
          setToken(regResult.token);
          localStorage.setItem('tasara_user', JSON.stringify(regResult.user));

          if (idUpload && idUpload.files.length) {
            var formData = new FormData();
            formData.append('idDocument', idUpload.files[0]);
            try { await fetch('/api/sellers/upload-id', { method: 'POST', headers: { Authorization: 'Bearer ' + regResult.token }, body: formData }); } catch(_) {}
          }

          window.location.href = 'dashboard-seller.html';
        } catch (apiErr) {
          err.textContent = apiErr.message || 'Registration failed. Please try again.';
          err.classList.add('show');
        }
      });
    }

    renderStep();
  }

  // ── Dashboards ──────────────────────────────────────────
  if (page === 'dashboard-buyer') {
    if (!requireAuth()) return;
    (async function() {
      try {
        var user = await apiFetch('/users/me');
        var el = function(id) { return document.getElementById(id); };
        if (el('userName')) el('userName').textContent = user.full_name.split(' ')[0];
        if (el('rowName')) el('rowName').textContent = user.full_name || '—';
        if (el('rowEmail')) el('rowEmail').textContent = user.email || '—';
        if (el('rowPhone')) el('rowPhone').textContent = user.phone || '—';
        if (el('rowAddress')) el('rowAddress').textContent = user.address || '—';
      } catch(err) { console.error(err); clearAuth(); window.location.href = 'login.html'; }
    })();
    var joinBtn = document.getElementById('joinBtn');
    if (joinBtn) joinBtn.addEventListener('click', function() {
      joinBtn.classList.add('tilted');
      setTimeout(function() { window.open('https://t.me/TasaraHub', '_blank', 'noopener'); }, 480);
    });
  }

  if (page === 'dashboard-seller') {
    if (!requireAuth()) return;
    (async function() {
      try {
        var user = await apiFetch('/users/me');
        var el = function(id) { return document.getElementById(id); };
        if (el('userName')) el('userName').textContent = user.full_name.split(' ')[0];
        if (el('rowName')) el('rowName').textContent = user.full_name || '—';
        if (el('rowEmail')) el('rowEmail').textContent = user.email || '—';
        if (el('rowPhone')) el('rowPhone').textContent = user.phone || '—';
        if (el('rowAddress')) el('rowAddress').textContent = user.address || '—';
        var app = await apiFetch('/sellers/my-application');
        if (el('rowBizName')) el('rowBizName').textContent = app.business_name || '—';
        if (el('rowBizLoc')) el('rowBizLoc').textContent = app.business_location || '—';
        if (el('rowTier')) el('rowTier').textContent = 'Tier ' + app.tier + ' — ' + app.tier_name;
        if (el('rowStake')) el('rowStake').textContent = app.tier_stake || '—';
        if (el('rowOmni')) el('rowOmni').textContent = app.tier_omni || '—';
        if (el('rowFile')) el('rowFile').textContent = app.id_document_path ? 'Uploaded ✓' : 'Not yet uploaded';
        if (el('tierNameTag')) el('tierNameTag').textContent = app.status === 'approved' ? app.tier_name : app.tier_name + ' (pending)';
        var statusPill = document.getElementById('statusPill');
        var statusCopy = document.getElementById('statusCopy');
        if (app.status === 'approved') {
          statusPill.className = 'status-pill status-live';
          statusPill.innerHTML = '<span class="status-dot"></span> Approved — you\'re a verified Merchant Steward';
          statusCopy.textContent = "You're verified under the Iron Standard. Your Omni has been issued and you're now listed in the Directory.";
        } else if (app.status === 'rejected') {
          statusPill.style.background = 'rgba(162,59,46,0.12)';
          statusPill.style.color = '#A23B2E';
          statusPill.innerHTML = '<span class="status-dot" style="background:#A23B2E;"></span> Application not approved';
          statusCopy.textContent = 'An Administrator reviewed your application and it was not approved this time. Contact tasaraafrica@gmail.com for details.';
        }
      } catch(err) { console.error(err); clearAuth(); window.location.href = 'login.html'; }
    })();
    var joinBtn = document.getElementById('joinBtn');
    if (joinBtn) joinBtn.addEventListener('click', function() {
      joinBtn.classList.add('tilted');
      setTimeout(function() { window.open('https://t.me/TasaraHub', '_blank', 'noopener'); }, 480);
    });
  }

  if (page === 'dashboard-admin') {
    if (!requireAuth()) return;
    (async function() {
      try {
        var user = await apiFetch('/users/me');
        if (user.role !== 'admin') { alert('Access denied — admin only.'); clearAuth(); window.location.href = 'login.html'; return; }
        var data = await apiFetch('/admin/applications');
        var pending = data.pending;
        var decided = data.decided;
        document.getElementById('statPending').textContent = data.stats.pending;
        document.getElementById('statApproved').textContent = data.stats.approved;
        document.getElementById('statRejected').textContent = data.stats.rejected;
        document.getElementById('statOmni').textContent = data.stats.omniCommitted;

        var pendingEmpty = document.getElementById('pendingEmpty');
        pendingEmpty.style.display = pending.length ? 'none' : 'block';
        function renderPendingRow(a) {
          return '<div class="app-row" data-id="'+escapeHtml(a.id)+'">'+
            '<div class="app-main">'+
              '<div class="app-name">'+escapeHtml(a.business_name||'Unnamed')+' <span style="color:rgba(242,238,227,0.5);font-weight:400;">— '+escapeHtml(a.full_name||'Unknown')+'</span></div>'+
              '<div class="app-meta">'+escapeHtml(a.business_location||'—')+' · '+escapeHtml(a.email||'')+'</div>'+
              '<div class="app-tier">Tier '+escapeHtml(a.tier)+' — '+escapeHtml(a.tier_name)+' · '+escapeHtml(a.tier_stake)+' stake · '+escapeHtml(a.tier_omni)+'</div>'+
            '</div>'+
            '<div class="app-actions">'+
              '<button class="btn-approve" data-action="approve" data-id="'+escapeHtml(a.id)+'">Approve</button>'+
              '<button class="btn-reject" data-action="reject" data-id="'+escapeHtml(a.id)+'">Reject</button>'+
            '</div></div>';
        }
        function renderDecidedRow(a) {
          return '<div class="app-row">'+
            '<div class="app-main">'+
              '<div class="app-name">'+escapeHtml(a.business_name||'Unnamed')+' <span style="color:rgba(242,238,227,0.5);font-weight:400;">— '+escapeHtml(a.full_name||'Unknown')+'</span></div>'+
              '<div class="app-meta">Tier '+escapeHtml(a.tier)+' — '+escapeHtml(a.tier_name)+'</div>'+
            '</div>'+
            '<span class="decision-tag decision-'+escapeHtml(a.status)+'">'+(a.status==='approved'?'Approved':'Rejected')+'</span>'+
          '</div>';
        }

        document.getElementById('pendingList').innerHTML = pending.map(renderPendingRow).join('');

        var decidedEmpty = document.getElementById('decidedEmpty');
        decidedEmpty.style.display = decided.length ? 'none' : 'block';
        document.getElementById('decidedList').innerHTML = decided.map(renderDecidedRow).join('');

        document.getElementById('pendingList').addEventListener('click', async function(e) {
          var btn = e.target.closest('button');
          if (!btn) return;
          var id = btn.dataset.id;
          var action = btn.dataset.action;
          try {
            await apiFetch('/admin/decide', { method: 'POST', body: JSON.stringify({ id: id, action: action }) });
            var refreshed = await apiFetch('/admin/applications');
            document.getElementById('statPending').textContent = refreshed.stats.pending;
            document.getElementById('statApproved').textContent = refreshed.stats.approved;
            document.getElementById('statRejected').textContent = refreshed.stats.rejected;
            document.getElementById('statOmni').textContent = refreshed.stats.omniCommitted;
            var np = refreshed.pending;
            document.getElementById('pendingEmpty').style.display = np.length ? 'none' : 'block';
            document.getElementById('pendingList').innerHTML = np.map(renderPendingRow).join('');
            var nd = refreshed.decided;
            document.getElementById('decidedEmpty').style.display = nd.length ? 'none' : 'block';
            document.getElementById('decidedList').innerHTML = nd.map(renderDecidedRow).join('');
          } catch(err) { alert('Action failed: ' + err.message); }
        });
      } catch(err) { console.error(err); clearAuth(); window.location.href = 'login.html'; }
    })();
  }
});