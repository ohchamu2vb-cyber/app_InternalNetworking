<script>
	import { onMount } from "svelte";
	onMount(() => {

		const initialPosts = [
			{ id: 1, author: "鈴木 はるか", dept: "デザイン部", initials: "鈴", time: "18分前", text: "お昼に新しくできたカレー屋さんへ。スパイス好きにはたまらないおいしさでした🍛 #ランチ部", image: "https://images.unsplash.com/photo-1565557623262-b51c2513a641?auto=format&fit=crop&w=1000&q=80", likes: 8, comments: [{ id: 101, author: "中村 健", initials: "中", time: "12分前", text: "ここ気になってました！おすすめのメニューありますか？" }] },
			{ id: 2, author: "中村 健", dept: "営業部", initials: "中", time: "1時間前", text: "おすすめしてもらった本、読み終わりました！チームで共有したい気づきがたくさん。貸してくれた田中さん、ありがとう。 #おすすめ本", image: "", likes: 4 },
			{ id: 3, author: "田中 葵", dept: "開発部", initials: "田", time: "3時間前", text: "今週のリリース、無事完了しました。関わってくれたみなさんおつかれさまでした！", image: "https://images.unsplash.com/photo-1522071820081-009f0129c71c?auto=format&fit=crop&w=1000&q=80", likes: 12 }
		];
		const initialListings = [
			{ id: 1, title: "デスクライト（ほぼ未使用）", category: "オフィス用品", description: "引っ越しで余ってしまいました。USB給電、明るさ調整できます。", owner: "山本 亮", initials: "山", image: "https://images.unsplash.com/photo-1507473885765-e6ed057f782c?auto=format&fit=crop&w=900&q=80", likes: 3 },
			{ id: 2, title: "デザインの基本 3冊セット", category: "本・学び", description: "読み終わったのでお譲りします。書き込みはありません。", owner: "鈴木 はるか", initials: "鈴", image: "https://images.unsplash.com/photo-1544947950-fa07a98d237f?auto=format&fit=crop&w=900&q=80", likes: 5 },
			{ id: 3, title: "観葉植物の挿し木", category: "暮らし", description: "育てやすいポトスです。小さな鉢ごとお渡しできます。", owner: "中村 健", initials: "中", image: "https://images.unsplash.com/photo-1485955900006-10f4d324d411?auto=format&fit=crop&w=900&q=80", likes: 2 },
			{ id: 4, title: "ホワイトボード用マーカー", category: "オフィス用品", description: "色がたくさんあります。会議室前の棚に置いておきます。", owner: "田中 葵", initials: "田", image: "https://images.unsplash.com/photo-1516321318423-f06f85e504b3?auto=format&fit=crop&w=900&q=80", likes: 1 }
		];
		const initialThanks = [
			{ to: "鈴木 はるか", stamp: "👏", message: "急な相談だったのに、資料づくりを手伝ってくれて本当に助かりました！", from: "中村 健", initials: "中", time: "今日 10:24" },
			{ to: "中村 健", stamp: "✨", message: "お客さまの声をすぐ共有してくれてありがとう。次の企画に活かせそうです。", from: "佐藤 みな", initials: "佐", time: "今日 09:10" },
			{ to: "田中 葵", stamp: "☕", message: "リリース対応おつかれさま！いつも丁寧なレビューに感謝しています。", from: "山本 亮", initials: "山", time: "昨日" },
			{ to: "山本 亮", stamp: "💐", message: "新メンバーの歓迎ランチ、段取りありがとう。楽しい時間でした！", from: "鈴木 はるか", initials: "鈴", time: "昨日" }
		];
		const readStore = (key, fallback) => { try { const value = JSON.parse(localStorage.getItem(key)); return value ?? fallback; } catch { return fallback; } };
		let posts = readStore("yorimichi-posts", initialPosts);
		let listings = readStore("yorimichi-listings", initialListings);
		let thanks = readStore("yorimichi-thanks", initialThanks);
		let selectedStamp = "👏";
		let selectedMedia = null;
		let selectedListingImage = null;
		const persist = (key, value) => { try { localStorage.setItem(key, JSON.stringify(value)); } catch { notify("保存容量を超えました。画像を小さくしてお試しください。"); } };
		const escapeHtml = (value) => String(value).replace(/[&<>"']/g, char => ({ "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;" })[char]);
		const notify = (message) => { const toast = document.getElementById("toast"); toast.textContent = message; toast.classList.add("show"); clearTimeout(notify.timer); notify.timer = setTimeout(() => toast.classList.remove("show"), 2600); };
		function switchView(name) {
			document.querySelectorAll(".view").forEach(view => view.classList.toggle("active", view.id === `view-${name}`));
			document.querySelectorAll("[data-view]").forEach(button => button.classList.toggle("active", button.dataset.view === name && button.classList.contains("nav-item")));
			window.scrollTo({ top: 0, behavior: "smooth" });
		}
		document.querySelectorAll("[data-view]").forEach(button => button.addEventListener("click", () => switchView(button.dataset.view)));
		document.getElementById("notificationButton").addEventListener("click", () => notify("新しい通知はありません"));
		function renderPosts() {
			const list = document.getElementById("postList");
			const query = document.getElementById("boardSearch").value.trim().toLowerCase();
			const terms = query.split(/\s+/).filter(Boolean);
			const filteredPosts = posts.filter(post => {
				const searchableText = `${post.text} ${post.author} ${post.dept || ""}`.toLowerCase();
				return terms.every(term => searchableText.includes(term));
			});
			document.getElementById("boardResultCount").textContent = query ? `「${query}」の検索結果：${filteredPosts.length}件` : `${filteredPosts.length}件の投稿`;
			list.innerHTML = filteredPosts.map(post => {
				const comments = post.comments || [];
				const commentThread = comments.length ? `<div class="comment-thread" aria-label="コメント一覧">${comments.map(comment => `<div class="comment-item"><div class="avatar small">${escapeHtml(comment.initials || "佐")}</div><div class="comment-content"><div class="comment-author">${escapeHtml(comment.author)}<span class="comment-time">${escapeHtml(comment.time || "たった今")}</span></div><p class="comment-text">${escapeHtml(comment.text)}</p></div></div>`).join("")}</div>` : "";
				return `<article class="post-card" data-post-id="${post.id}"><div class="post-head"><div class="avatar">${escapeHtml(post.initials || "佐")}</div><div><div class="post-author">${escapeHtml(post.author)}</div><div class="post-dept">${escapeHtml(post.dept || "企画チーム")} ・ ${escapeHtml(post.time)}</div></div></div><p class="post-text">${escapeHtml(post.text)}</p>${post.image ? (post.image.startsWith("data:video") ? `<video class="post-media" src="${post.image}" controls></video>` : `<img class="post-media" src="${post.image}" alt="投稿に添付された画像">`) : ""}<div class="post-actions"><button class="post-action ${post.liked ? "liked" : ""}" data-like-post="${post.id}">♡ いいね ${post.likes || 0}</button><button class="post-action" data-toggle-comments="${post.id}">コメント ${comments.length ? `(${comments.length})` : ""}</button></div>${commentThread}<form class="comment-form" data-comment-form="${post.id}"><div class="avatar small">佐</div><textarea class="comment-input" name="comment" rows="1" maxlength="300" placeholder="${escapeHtml(post.author)}さんの投稿にコメント" aria-label="${escapeHtml(post.author)}さんの投稿にコメント"></textarea><button class="comment-submit" type="submit">返信</button></form></article>`;
			}).join("") || `<div class="empty-state">${query ? "一致する投稿がありません。別のキーワードやハッシュタグをお試しください。" : "まだ投稿はありません。最初の投稿をしてみましょう。"}</div>`;
			list.querySelectorAll("[data-like-post]").forEach(button => button.addEventListener("click", () => { const post = posts.find(item => item.id === Number(button.dataset.likePost)); post.liked = !post.liked; post.likes = Math.max(0, (post.likes || 0) + (post.liked ? 1 : -1)); persist("yorimichi-posts", posts); renderPosts(); }));
			list.querySelectorAll("[data-toggle-comments]").forEach(button => button.addEventListener("click", () => {
				const form = list.querySelector(`[data-comment-form="${button.dataset.toggleComments}"]`);
				form.querySelector("textarea").focus();
			}));
			list.querySelectorAll("[data-comment-form]").forEach(form => form.addEventListener("submit", event => {
				event.preventDefault();
				const text = form.elements.comment.value.trim();
				if (!text) { notify("コメントを入力してください"); return; }
				const post = posts.find(item => item.id === Number(form.dataset.commentForm));
				post.comments ||= [];
				post.comments.push({ id: Date.now(), author: "佐藤 みな", initials: "佐", time: "たった今", text });
				persist("yorimichi-posts", posts);
				renderPosts();
				notify("元の投稿にコメントしました");
			}));
			const tagCounts = new Map();
			posts.forEach(post => (post.text.match(/#[\p{L}\p{N}_-]+/gu) || []).forEach(tag => tagCounts.set(tag, (tagCounts.get(tag) || 0) + 1)));
			const tags = Array.from(tagCounts, ([tag, count]) => ({ tag, count })).sort((a, b) => b.count - a.count || a.tag.localeCompare(b.tag, "ja"));
			const tagList = document.getElementById("boardTags");
			tagList.innerHTML = tags.length ? tags.map(({ tag, count }) => `<button class="tag-chip" type="button" data-search-tag="${escapeHtml(tag)}">${escapeHtml(tag)} <span>${count}</span></button>`).join("") : `<span class="tag-empty">投稿にハッシュタグを付けると、ここから探せます。</span>`;
			tagList.querySelectorAll("[data-search-tag]").forEach(button => button.addEventListener("click", () => {
				document.getElementById("boardSearch").value = button.dataset.searchTag;
				renderPosts();
			}));
			document.getElementById("homeFeed").innerHTML = posts.slice(0, 2).map(post => `<div class="preview-post"><div class="avatar small">${escapeHtml(post.initials || "佐")}</div><div><p>${escapeHtml(post.text)}</p><div class="preview-meta">${escapeHtml(post.author)} ・ ${escapeHtml(post.time)}</div></div></div>`).join("");
		}
		function renderListings() {
			const query = document.getElementById("listingSearch").value.toLowerCase();
			const category = document.getElementById("categoryFilter").value;
			const results = listings.filter(item => (category === "all" || item.category === category) && `${item.title} ${item.description} ${item.category}`.toLowerCase().includes(query));
			const grid = document.getElementById("listingGrid");
			grid.innerHTML = results.map(item => `<article class="listing-card"><div class="listing-image">${item.image ? `<img src="${item.image}" alt="${escapeHtml(item.title)}">` : ""}<span class="category-tag">${escapeHtml(item.category)}</span><span class="free-tag">無料でゆずります</span></div><div class="listing-body"><h3>${escapeHtml(item.title)}</h3><p>${escapeHtml(item.description)}</p><div class="listing-foot"><span class="listing-owner"><span class="avatar small">${escapeHtml(item.initials || "佐")}</span>${escapeHtml(item.owner)}</span><span class="listing-actions"><button class="icon-action ${item.liked ? "liked" : ""}" data-like-listing="${item.id}" aria-label="いいね">♡ ${item.likes || 0}</button><button class="listing-open" type="button" data-listing-detail="${item.id}">詳細を見る →</button></span></div></div></article>`).join("") || `<div class="empty-state" style="grid-column:1/-1">該当するアイテムはありません。</div>`;
			grid.querySelectorAll("[data-like-listing]").forEach(button => button.addEventListener("click", () => { const item = listings.find(entry => entry.id === Number(button.dataset.likeListing)); item.liked = !item.liked; item.likes = Math.max(0, (item.likes || 0) + (item.liked ? 1 : -1)); persist("yorimichi-listings", listings); renderListings(); }));
			grid.querySelectorAll("[data-listing-detail]").forEach(button => button.addEventListener("click", () => showListingDetail(Number(button.dataset.listingDetail))));
		}
		function showListingDetail(id) {
			const item = listings.find(listing => listing.id === id);
			if (!item) return;
			document.getElementById("listingDetailContent").innerHTML = `<div class="listing-detail"><div class="listing-detail-image">${item.image ? `<img src="${item.image}" alt="${escapeHtml(item.title)}">` : `<div class="empty-state" style="height:100%;min-height:300px;display:grid;place-items:center">写真は登録されていません</div>`}</div><div class="listing-detail-copy"><span class="category-tag" style="position:static;display:inline-block">${escapeHtml(item.category)}</span><h2>${escapeHtml(item.title)}</h2><p>${escapeHtml(item.description)}</p><div class="listing-detail-owner"><span class="avatar">${escapeHtml(item.initials || "佐")}</span><span>${escapeHtml(item.owner)}</span></div><button class="primary-button" type="button" id="detailContact">出品者に連絡する</button></div></div>`;
			document.getElementById("detailContact").addEventListener("click", () => notify(`${item.owner}さんに連絡したいことを伝えました`));
			switchView("listing-detail");
		}
		function renderThanks(animate = false) {
			document.getElementById("thanksCount").textContent = `${thanks.length}件`;
			document.getElementById("thanksGrid").innerHTML = thanks.map(item => `<article class="thanks-card"><div class="thanks-card-top"><span>${item.stamp}</span><span style="font-size:10px;color:#939d97">${escapeHtml(item.time || "たった今")}</span></div><p>${escapeHtml(item.message || "ありがとう！")}</p><div class="thanks-from"><div class="avatar small">${escapeHtml(item.initials || "佐")}</div><span>${escapeHtml(item.from)} から <strong>${escapeHtml(item.to)}</strong> へ</span></div></article>`).join("");
			const tree = document.getElementById("thanksTree");
			const stages = [
				{ min: 10, className: "mature", label: "大樹", caption: "たくさんの感謝が、豊かな木になりました。" },
				{ min: 6, className: "growing", label: "成長中", caption: "ありがとうが集まり、ぐんぐん成長中。" },
				{ min: 3, className: "young", label: "若木", caption: "感謝の気持ちが、木を育てています。" },
				{ min: 1, className: "sprout", label: "芽吹き", caption: "最初のありがとうが芽を出しました。" },
				{ min: 0, className: "seed", label: "これから", caption: "ありがとうを贈って、木を育てよう。" }
			];
			const stage = stages.find(item => thanks.length >= item.min);
			tree.className = `tree-scene tree-${stage.className}`;
			tree.setAttribute("aria-label", `サンクスの木: ${stage.label}、${thanks.length}件`);
			document.getElementById("thanksTotal").textContent = `${thanks.length}件`;
			document.getElementById("treeStage").textContent = stage.label;
			document.getElementById("treeCaption").textContent = stage.caption;
			if (animate) {
				tree.classList.remove("grow-pulse");
				void tree.offsetWidth;
				tree.classList.add("grow-pulse");
				setTimeout(() => tree.classList.remove("grow-pulse"), 900);
			}
		}
		document.getElementById("boardSearch").addEventListener("input", renderPosts);
		document.getElementById("clearBoardSearch").addEventListener("click", () => { document.getElementById("boardSearch").value = ""; renderPosts(); });
		document.getElementById("listingSearch").addEventListener("input", renderListings);
		document.getElementById("categoryFilter").addEventListener("change", renderListings);
		const listingDialog = document.getElementById("listingDialog");
		document.getElementById("openListingForm").addEventListener("click", () => listingDialog.showModal());
		document.getElementById("closeListingForm").addEventListener("click", () => listingDialog.close());
		document.getElementById("listingImageInput").addEventListener("change", event => {
			const file = event.target.files[0];
			if (!file) return;
			if (file.size > 2 * 1024 * 1024) { notify("写真は2MB以下にしてください"); event.target.value = ""; return; }
			const reader = new FileReader();
			reader.onload = () => {
				selectedListingImage = reader.result;
				document.getElementById("listingImagePreview").innerHTML = `<img src="${reader.result}" alt="出品写真のプレビュー" style="display:block;max-width:100%;max-height:180px;object-fit:contain;border-radius:6px;margin-top:8px">`;
			};
			reader.readAsDataURL(file);
		});
		document.getElementById("listingForm").addEventListener("submit", event => {
			event.preventDefault();
			listings.unshift({ id: Date.now(), title: document.getElementById("itemTitle").value.trim(), category: document.getElementById("itemCategory").value, description: document.getElementById("itemDescription").value.trim() || "詳しくは出品者にお問い合わせください。", owner: "佐藤 みな", initials: "佐", image: selectedListingImage || "", likes: 0 });
			persist("yorimichi-listings", listings); renderListings(); event.target.reset(); selectedListingImage = null; document.getElementById("listingImagePreview").innerHTML = ""; listingDialog.close(); notify("アイテムを出品しました");
		});
		document.getElementById("mediaInput").addEventListener("change", event => {
			const file = event.target.files[0]; if (!file) return;
			if (file.size > 2 * 1024 * 1024) { notify("添付ファイルは2MB以下にしてください"); event.target.value = ""; return; }
			const reader = new FileReader(); reader.onload = () => { selectedMedia = { data: reader.result, type: file.type }; document.getElementById("mediaPreview").innerHTML = file.type.startsWith("video/") ? `<video class="post-media" src="${reader.result}" controls></video>` : `<img class="post-media" style="max-height:150px;margin:10px 0 0 43px;width:calc(100% - 43px)" src="${reader.result}" alt="添付プレビュー">`; }; reader.readAsDataURL(file);
		});
		document.getElementById("postForm").addEventListener("submit", event => {
			event.preventDefault(); const text = document.getElementById("postText").value.trim();
			if (!text && !selectedMedia) { notify("投稿内容か写真・動画を追加してください"); return; }
			posts.unshift({ id: Date.now(), author: "佐藤 みな", dept: "企画チーム", initials: "佐", time: "たった今", text, image: selectedMedia?.data || "", likes: 0, comments: [] });
			persist("yorimichi-posts", posts); event.target.reset(); selectedMedia = null; document.getElementById("mediaPreview").innerHTML = ""; renderPosts(); notify("掲示板に投稿しました");
		});
		document.getElementById("stampRow").addEventListener("click", event => { const button = event.target.closest("[data-stamp]"); if (!button) return; selectedStamp = button.dataset.stamp; document.querySelectorAll(".stamp").forEach(stamp => stamp.classList.toggle("selected", stamp === button)); });
		document.getElementById("thanksForm").addEventListener("submit", event => {
			event.preventDefault(); const to = document.getElementById("thanksTo").value; const message = document.getElementById("thanksMessage").value.trim();
			if (!to) { notify("届ける相手を選んでください"); return; }
			if (!message) { notify("メッセージを入力してください"); return; }
			thanks.unshift({ to: to.split("（")[0], stamp: selectedStamp, message, from: "佐藤 みな", initials: "佐", time: "たった今" }); persist("yorimichi-thanks", thanks); renderThanks(true); event.target.reset(); notify(`${to.split("（")[0]}さんにありがとうを届けました`);
		});
		renderPosts(); renderListings(); renderThanks();
	
	});
</script>

<svelte:head>
	<title>よりみち | 社内コミュニティ</title>
	<meta name="description" content="モノも、アイデアも、ありがとうも。社内の「ちょっといい」をここから。" />
</svelte:head>

<div class="app">
		<header class="topbar">
			<button class="brand" type="button" data-view="home" aria-label="ホームへ">
				<span class="brand-mark">よ</span>
				<span><span class="brand-name">よりみち</span><span class="brand-sub">INTERNAL COMMUNITY</span></span>
			</button>
			<div class="top-actions">
				<button class="text-button" type="button" aria-label="通知" id="notificationButton">通知 <span aria-hidden="true">◦</span></button>
				<div class="profile-mini"><div class="avatar small">佐</div><div><div class="profile-name">佐藤 みな</div><div class="profile-dept">企画チーム</div></div></div>
			</div>
		</header>
		<div class="layout">
			<aside class="sidebar">
				<p class="side-label">MENU</p>
				<nav class="nav-list" aria-label="メインメニュー">
					<button class="nav-item active" data-view="home"><span class="nav-icon">⌂</span>ホーム</button>
					<button class="nav-item" data-view="giveaway"><span class="nav-icon">↗</span>ゆずる・もらう</button>
					<button class="nav-item" data-view="board"><span class="nav-icon">▤</span>みんなの掲示板</button>
					<button class="nav-item" data-view="thanks"><span class="nav-icon">♡</span>サンクス</button>
				</nav>
				<div class="side-note"><strong>つながりを、もっと気軽に。</strong>モノも、アイデアも、ありがとうも。社内の「ちょっといい」をここから。</div>
			</aside>
			<main>
				<section class="view active" id="view-home">
					<div class="home-hero">
						<div class="hero-copy"><p class="eyebrow">GOOD THINGS HAPPEN HERE</p><h1>今日も、誰かと<br>いいよりみち。</h1><p>モノをゆずったり、日々のことをシェアしたり。</p><button class="primary-button" type="button" data-view="board">みんなの投稿を見る <span>→</span></button></div>
						<div class="hero-illustration"><img src="https://images.unsplash.com/photo-1521737711867-e3b97375f902?auto=format&fit=crop&w=600&q=80" alt="オフィスで会話する同僚たち"><div class="hero-tag">社内のつながり、ここから</div></div>
					</div>
					<div class="section-title"><h2>できること</h2><span style="color:var(--muted);font-size:11px">気になる場所からのぞいてみよう</span></div>
					<div class="feature-grid">
						<button class="feature-card" data-view="giveaway"><span class="feature-top"><span class="feature-symbol green">↗</span><span class="feature-arrow">↗</span></span><h3>ゆずる・もらう</h3><p>使わなくなったモノを、必要な人へ。</p></button>
						<button class="feature-card" data-view="board"><span class="feature-top"><span class="feature-symbol orange">▤</span><span class="feature-arrow">↗</span></span><h3>みんなの掲示板</h3><p>仕事のことも、日々の小さな発見も。</p></button>
						<button class="feature-card" data-view="thanks"><span class="feature-top"><span class="feature-symbol blue">♡</span><span class="feature-arrow">↗</span></span><h3>サンクス</h3><p>「ありがとう」を、ちゃんと届けよう。</p></button>
					</div>
					<div class="home-lower">
						<div class="feed-preview"><div class="section-title"><h2>掲示板の新着</h2><button class="link-button" data-view="board">すべて見る →</button></div><div id="homeFeed"></div></div>
						<div class="thanks-highlight"><div class="section-title"><h2>サンクスの木</h2><button class="link-button" data-view="thanks">ありがとうを送る →</button></div><div class="tree-layout"><div class="tree-scene tree-sprout" id="thanksTree" role="img" aria-label="サンクスの木"><div class="tree-ground"></div><div class="tree-trunk"></div><div class="tree-canopy"><i class="tree-leaf"></i><i class="tree-leaf"></i><i class="tree-leaf"></i><i class="tree-leaf"></i></div></div><div class="tree-copy"><p class="eyebrow">THANKS GROWN</p><strong class="tree-count" id="thanksTotal">0件</strong><span class="tree-stage" id="treeStage">種まき前</span><p class="tree-caption" id="treeCaption">ありがとうを贈って、木を育てよう。</p></div></div></div>
					</div>
				</section>

				<section class="view" id="view-giveaway">
					<div class="view-head"><div><p class="eyebrow">PASS IT ALONG</p><h1>ゆずる・もらう</h1><p class="lead">まだ使えるものを、必要としている人へ。</p></div><button class="primary-button" id="openListingForm">＋ <span>出品する</span></button></div>
					<div class="toolbar"><input class="searchbox" id="listingSearch" type="search" placeholder="アイテムを検索"><select class="filter-select" id="categoryFilter" aria-label="カテゴリ"><option value="all">すべてのカテゴリ</option><option>オフィス用品</option><option>本・学び</option><option>暮らし</option><option>その他</option></select></div>
					<div class="listing-grid" id="listingGrid"></div>
				</section>
				<section class="view" id="view-listing-detail">
					<div class="view-head"><div><p class="eyebrow">ITEM DETAILS</p><h1>アイテムの詳細</h1></div><button class="secondary-button" type="button" data-view="giveaway">← 一覧に戻る</button></div>
					<div id="listingDetailContent"></div>
				</section>

				<section class="view" id="view-board">
					<div class="view-head"><div><p class="eyebrow">OPEN BOARD</p><h1>みんなの掲示板</h1><p class="lead">近況やアイデアを気軽にシェアしよう。</p></div></div>
					<div class="board-search"><input class="searchbox" id="boardSearch" type="search" placeholder="キーワードまたは #ハッシュタグで検索" aria-label="掲示板を検索"><button class="search-clear" type="button" id="clearBoardSearch">クリア</button><span class="board-result-count" id="boardResultCount" aria-live="polite"></span></div>
					<div class="board-tags" id="boardTags"></div>
					<div class="board-layout"><div>
						<form class="composer" id="postForm"><div class="composer-row"><div class="avatar">佐</div><textarea class="textarea" id="postText" placeholder="みんなにシェアしたいことは？" aria-label="投稿内容" maxlength="500"></textarea></div><div id="mediaPreview"></div><div class="composer-tools"><label class="upload-label" for="mediaInput"><span>▧</span> 写真・動画を追加</label><input id="mediaInput" type="file" accept="image/*,video/*"><button class="primary-button" type="submit">投稿する</button></div></form>
						<div class="post-list" id="postList"></div>
					</div><aside class="board-aside"><h3>ハッシュタグ検索</h3><p style="margin:0;color:var(--muted);font-size:11px;line-height:1.7">投稿に付いたタグをタップすると、そのタグの投稿だけを表示します。</p></aside></div>
				</section>

				<section class="view" id="view-thanks">
					<div class="view-head"><div><p class="eyebrow">A LITTLE THANK YOU</p><h1>サンクス</h1><p class="lead">助けてもらったこと、うれしかったこと。ひとこと届けてみよう。</p></div></div>
					<div class="thanks-compose"><form id="thanksForm"><div class="thanks-form">
						<div class="form-field"><label for="thanksTo">届ける相手</label><select id="thanksTo" required><option value="">相手を選択</option><option>鈴木 はるか（デザイン部）</option><option>中村 健（営業部）</option><option>田中 葵（開発部）</option><option>山本 亮（人事部）</option><option>佐藤 みな（企画チーム）</option></select></div>
						<div class="form-field"><label>気持ちをスタンプで</label><div class="stamp-row" id="stampRow"><button type="button" class="stamp selected" data-stamp="👏"><span>👏</span>助かりました</button><button type="button" class="stamp" data-stamp="✨"><span>✨</span>さすがです</button><button type="button" class="stamp" data-stamp="💐"><span>💐</span>ありがとう</button><button type="button" class="stamp" data-stamp="☕"><span>☕</span>おつかれさま</button></div></div>
						<div class="form-field full"><label for="thanksMessage">メッセージ</label><textarea class="textarea" id="thanksMessage" placeholder="どんなことがうれしかったか、伝えてみましょう。" maxlength="240"></textarea></div>
					</div><div class="thanks-submit"><button class="primary-button" type="submit">♡ 感謝を届ける</button></div></form></div>
					<div class="thanks-wall-title"><h2>みんなのサンクス</h2><span style="font-size:11px;color:var(--muted)" id="thanksCount"></span></div><div class="thanks-grid" id="thanksGrid"></div>
				</section>
			</main>
		</div>
		<nav class="mobile-nav" aria-label="メインメニュー">
			<button class="nav-item active" data-view="home"><span class="nav-icon">⌂</span>ホーム</button><button class="nav-item" data-view="giveaway"><span class="nav-icon">↗</span>ゆずる</button><button class="nav-item" data-view="board"><span class="nav-icon">▤</span>掲示板</button><button class="nav-item" data-view="thanks"><span class="nav-icon">♡</span>サンクス</button>
		</nav>
		<div class="toast" id="toast" role="status" aria-live="polite"></div>
	</div>
	<dialog id="listingDialog" style="border:0;border-radius:9px;padding:0;width:min(480px,calc(100% - 32px));box-shadow:0 20px 70px #1d302833">
		<form id="listingForm" style="padding:22px;display:grid;gap:14px"><div style="display:flex;justify-content:space-between;align-items:center"><h2 style="margin:0;font-size:18px">アイテムを出品</h2><button type="button" id="closeListingForm" class="text-button" aria-label="閉じる">✕</button></div><div class="form-field"><label for="itemTitle">アイテム名</label><input id="itemTitle" class="field" required maxlength="45" placeholder="例：使わなくなったマグカップ"></div><div class="form-field"><label for="itemCategory">カテゴリ</label><select id="itemCategory"><option>オフィス用品</option><option>本・学び</option><option>暮らし</option><option>その他</option></select></div><div class="form-field"><label for="itemDescription">説明</label><textarea id="itemDescription" class="textarea" style="border:1px solid var(--line)" maxlength="180" placeholder="状態や受け渡し方法など"></textarea></div><div class="form-field"><label for="listingImageInput">写真</label><label class="secondary-button image-picker" for="listingImageInput">写真を選ぶ</label><input id="listingImageInput" type="file" accept="image/*"><div id="listingImagePreview"></div></div><button class="primary-button" type="submit">出品する</button></form>
	</dialog>
