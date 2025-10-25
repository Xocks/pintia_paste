// ==UserScript==
// @name         Pintia 自动剪切板粘贴器
// @namespace    http://tampermonkey.net/
// @version      3.4
// @description  在 Pintia.cn 自动读取剪切板并逐字符粘贴文本，支持历史记录和自动刷新
// @author       Bzjdtc
// @match        https://pintia.cn/*
// @icon         data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==
// @grant        GM_setClipboard
// @grant        GM_getClipboard
// @grant        GM_setValue
// @grant        GM_getValue
// ==/UserScript==

(function() {
    'use strict';

    // 添加样式到页面
    const style = document.createElement('style');
    style.textContent = `
        .pintia-paster-container {
            position: fixed;
            bottom: 10px;
            right: 20px;
            width: 240px;
            height: 300px;
            background-color: #ffffff;
            border: 2px solid #4a90e2;
            padding: 12px;
            border-radius: 12px;
            box-shadow: 0 8px 30px rgba(0,0,0,0.15);
            z-index: 10000;
            font-family: Arial, sans-serif;
            background: linear-gradient(135deg, #f5f7ff 0%, #f0f4ff 100%);
            transition: box-shadow 0.3s ease, border-color 0.3s ease;
            cursor: move;
            will-change: transform;
            display: flex;
            flex-direction: column;
        }

        .pintia-paster-container.dragging {
            box-shadow: 0 12px 40px rgba(0,0,0,0.2);
            border-color: #3a7bc8;
            cursor: grabbing;
            transition: none;
        }

        .pintia-clipboard-preview {
            width: 100%;
            height: 60px;
            padding: 8px;
            font-size: 12px;
            line-height: 1.4;
            resize: none;
            overflow: auto;
            box-sizing: border-box;
            border: 1px solid #d1d9e6;
            border-radius: 6px;
            background-color: #f8f9fa;
            font-family: monospace, Consolas, Monaco;
            outline: none;
            cursor: default;
            color: #666;
            margin-bottom: 8px;
        }

        .pintia-clipboard-preview::placeholder {
            color: #999;
            font-style: italic;
        }

        .pintia-history-container {
            margin-bottom: 8px;
        }

        .pintia-history-select {
            width: 100%;
            padding: 6px 8px;
            font-size: 11px;
            border: 1px solid #d1d9e6;
            border-radius: 4px;
            background-color: white;
        }

        .pintia-paster-button-container {
            display: flex;
            flex-wrap: wrap;
            gap: 6px;
            margin-bottom: 8px;
        }

        .pintia-paster-button {
            flex: 1;
            min-width: 70px;
            height: 28px;
            padding: 4px 8px;
            background-color: #4a90e2;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-weight: 600;
            font-size: 11px;
            transition: all 0.2s ease;
        }

        .pintia-paster-button:hover:not(:disabled) {
            background-color: #3a7bc8;
        }

        .pintia-paster-button:active:not(:disabled) {
            transform: translateY(1px);
        }

        .pintia-paster-button:disabled {
            background-color: #a0a0a0;
            cursor: not-allowed;
            transform: none;
        }

        .pintia-paster-button.read-clipboard {
            background-color: #27ae60;
        }

        .pintia-paster-button.read-clipboard:hover:not(:disabled) {
            background-color: #219653;
        }

        .pintia-paster-button.clear {
            background-color: #e74c3c;
        }

        .pintia-paster-button.clear:hover:not(:disabled) {
            background-color: #c0392b;
        }

        .pintia-settings-container {
            display: flex;
            flex-direction: column;
            gap: 6px;
            margin-bottom: 8px;
        }

        .pintia-setting-row {
            display: flex;
            align-items: center;
            justify-content: space-between;
        }

        .pintia-mode-label, .pintia-speed-label, .pintia-refresh-label {
            font-size: 11px;
            color: #666;
            min-width: 40px;
        }

        .pintia-mode-select, .pintia-speed-select, .pintia-refresh-input {
            padding: 4px 6px;
            border: 1px solid #d1d9e6;
            border-radius: 4px;
            font-size: 11px;
            background-color: white;
            width: 120px;
        }

        .pintia-refresh-input {
            width: 60px;
        }

        .pintia-auto-refresh-row {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-top: 4px;
        }

        .pintia-auto-refresh-label {
            font-size: 11px;
            color: #666;
        }

        .pintia-auto-refresh-checkbox {
            margin-right: 6px;
        }

        .pintia-progress-container {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-top: 4px;
            display: none;
        }

        .pintia-status-text {
            font-size: 11px;
            color: #4a90e2;
            font-weight: 600;
            flex: 1;
            text-align: center;
        }

        .pintia-cancel-button {
            width: 50px;
            height: 22px;
            padding: 2px 6px;
            background-color: #4a90e2;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-weight: 600;
            font-size: 10px;
            margin-left: 8px;
        }

        .pintia-cancel-button:hover:not(:disabled) {
            background-color: #3a7bc8;
        }
    `;
    document.head.appendChild(style);

    // 创建容器
    const container = document.createElement('div');
    container.className = 'pintia-paster-container';

    // 创建剪切板内容预览框
    const clipboardPreview = document.createElement('textarea');
    clipboardPreview.className = 'pintia-clipboard-preview';
    clipboardPreview.placeholder = '剪切板内容将显示在这里...';
    clipboardPreview.readOnly = true;

    // 创建历史记录容器
    const historyContainer = document.createElement('div');
    historyContainer.className = 'pintia-history-container';

    const historySelect = document.createElement('select');
    historySelect.className = 'pintia-history-select';
    historySelect.innerHTML = '<option value="">-- 选择历史记录 --</option>';

    historyContainer.appendChild(historySelect);

    // 创建按钮容器
    const buttonContainer = document.createElement('div');
    buttonContainer.className = 'pintia-paster-button-container';

    // 创建读取剪切板按钮
    const readClipboardButton = document.createElement('button');
    readClipboardButton.className = 'pintia-paster-button read-clipboard';
    readClipboardButton.textContent = '读取剪切板';

    // 创建开始粘贴按钮
    const pasteButton = document.createElement('button');
    pasteButton.className = 'pintia-paster-button';
    pasteButton.textContent = '开始粘贴';

    // 创建清空按钮
    const clearButton = document.createElement('button');
    clearButton.className = 'pintia-paster-button clear';
    clearButton.textContent = '清空';

    buttonContainer.appendChild(readClipboardButton);
    buttonContainer.appendChild(pasteButton);
    buttonContainer.appendChild(clearButton);

    // 创建设置容器
    const settingsContainer = document.createElement('div');
    settingsContainer.className = 'pintia-settings-container';

    // 创建模式选择行
    const modeRow = document.createElement('div');
    modeRow.className = 'pintia-setting-row';

    const modeLabel = document.createElement('span');
    modeLabel.className = 'pintia-mode-label';
    modeLabel.textContent = '模式:';

    const modeSelect = document.createElement('select');
    modeSelect.className = 'pintia-mode-select';

    const appendOption = document.createElement('option');
    appendOption.value = 'append';
    appendOption.textContent = '追加';

    const replaceOption = document.createElement('option');
    replaceOption.value = 'replace';
    replaceOption.textContent = '替换';

    modeSelect.appendChild(appendOption);
    modeSelect.appendChild(replaceOption);

    modeRow.appendChild(modeLabel);
    modeRow.appendChild(modeSelect);

    // 创建速度选择行
    const speedRow = document.createElement('div');
    speedRow.className = 'pintia-setting-row';

    const speedLabel = document.createElement('span');
    speedLabel.className = 'pintia-speed-label';
    speedLabel.textContent = '速度:';

    const speedSelect = document.createElement('select');
    speedSelect.className = 'pintia-speed-select';

    const ultraFastOption = document.createElement('option');
    ultraFastOption.value = 'ultra-fast';
    ultraFastOption.textContent = '超高速';

    const fastOption = document.createElement('option');
    fastOption.value = 'fast';
    fastOption.textContent = '高速';

    const mediumOption = document.createElement('option');
    mediumOption.value = 'medium';
    mediumOption.textContent = '中速';

    const slowOption = document.createElement('option');
    slowOption.value = 'slow';
    slowOption.textContent = '慢速';

    speedSelect.appendChild(ultraFastOption);
    speedSelect.appendChild(fastOption);
    speedSelect.appendChild(mediumOption);
    speedSelect.appendChild(slowOption);
    speedSelect.value = 'ultra-fast'; // 默认设置为超高速

    speedRow.appendChild(speedLabel);
    speedRow.appendChild(speedSelect);

    // 创建自动刷新设置行
    const refreshRow = document.createElement('div');
    refreshRow.className = 'pintia-auto-refresh-row';

    const refreshLabel = document.createElement('span');
    refreshLabel.className = 'pintia-auto-refresh-label';
    refreshLabel.textContent = '自动刷新:';

    const refreshCheckbox = document.createElement('input');
    refreshCheckbox.className = 'pintia-auto-refresh-checkbox';
    refreshCheckbox.type = 'checkbox';

    const refreshInput = document.createElement('input');
    refreshInput.className = 'pintia-refresh-input';
    refreshInput.type = 'number';
    refreshInput.min = '1';
    refreshInput.max = '60';
    refreshInput.value = '5';
    refreshInput.placeholder = '秒';

    refreshRow.appendChild(refreshLabel);
    refreshRow.appendChild(refreshCheckbox);
    refreshRow.appendChild(refreshInput);

    // 将设置行添加到设置容器
    settingsContainer.appendChild(modeRow);
    settingsContainer.appendChild(speedRow);
    settingsContainer.appendChild(refreshRow);

    // 创建进度容器
    const progressContainer = document.createElement('div');
    progressContainer.className = 'pintia-progress-container';

    const statusText = document.createElement('div');
    statusText.className = 'pintia-status-text';

    const cancelButton = document.createElement('button');
    cancelButton.className = 'pintia-cancel-button';
    cancelButton.textContent = '取消';

    progressContainer.appendChild(statusText);
    progressContainer.appendChild(cancelButton);

    // 组装容器
    container.appendChild(clipboardPreview);
    container.appendChild(historyContainer);
    container.appendChild(buttonContainer);
    container.appendChild(settingsContainer);
    container.appendChild(progressContainer);
    document.body.appendChild(container);

    // === 丝滑拖拽实现 ===
    let isDragging = false;
    let dragStartX, dragStartY;
    let containerStartX, containerStartY;

    function getContainerPosition() {
        const rect = container.getBoundingClientRect();
        return {
            x: rect.left,
            y: rect.top
        };
    }

    function setContainerPosition(x, y) {
        const maxX = window.innerWidth - container.offsetWidth;
        const maxY = window.innerHeight - container.offsetHeight;

        const constrainedX = Math.max(0, Math.min(x, maxX));
        const constrainedY = Math.max(0, Math.min(y, maxY));

        container.style.left = constrainedX + 'px';
        container.style.top = constrainedY + 'px';
        container.style.right = 'auto';
        container.style.bottom = 'auto';
    }

    function startDrag(e) {
        if (e.target === clipboardPreview || e.target === historySelect || 
            e.target === readClipboardButton || e.target === pasteButton || 
            e.target === clearButton || e.target === modeSelect || 
            e.target === speedSelect || e.target === refreshCheckbox ||
            e.target === refreshInput || e.target === cancelButton) {
            return;
        }

        isDragging = true;
        const pos = getContainerPosition();
        containerStartX = pos.x;
        containerStartY = pos.y;
        dragStartX = e.clientX;
        dragStartY = e.clientY;

        container.classList.add('dragging');
        e.preventDefault();
    }

    function onDrag(e) {
        if (!isDragging) return;

        const deltaX = e.clientX - dragStartX;
        const deltaY = e.clientY - dragStartY;

        const newX = containerStartX + deltaX;
        const newY = containerStartY + deltaY;

        setContainerPosition(newX, newY);
        e.preventDefault();
    }

    function endDrag() {
        if (!isDragging) return;

        isDragging = false;
        container.classList.remove('dragging');
    }

    container.addEventListener('mousedown', startDrag);
    document.addEventListener('mousemove', onDrag);
    document.addEventListener('mouseup', endDrag);

    // 触摸设备支持
    container.addEventListener('touchstart', (e) => {
        if (e.target === clipboardPreview || e.target === historySelect || 
            e.target === readClipboardButton || e.target === pasteButton || 
            e.target === clearButton || e.target === modeSelect || 
            e.target === speedSelect || e.target === refreshCheckbox ||
            e.target === refreshInput || e.target === cancelButton) {
            return;
        }
        startDrag(e.touches[0]);
    });

    document.addEventListener('touchmove', (e) => {
        onDrag(e.touches[0]);
    });

    document.addEventListener('touchend', endDrag);

    window.addEventListener('resize', () => {
        const pos = getContainerPosition();
        setContainerPosition(pos.x, pos.y);
    });

    // === 剪切板功能 ===
    let clipboardText = '';
    let isTyping = false;
    let cancelTyping = false;
    let clipboardHistory = [];
    let autoRefreshInterval = null;
    const MAX_HISTORY_ITEMS = 10;

    // 初始化历史记录
    function initHistory() {
        try {
            const savedHistory = GM_getValue('clipboardHistory', []);
            clipboardHistory = Array.isArray(savedHistory) ? savedHistory : [];
            updateHistorySelect();
        } catch (e) {
            console.error('读取历史记录失败:', e);
            clipboardHistory = [];
        }
    }

    // 更新历史记录选择框
    function updateHistorySelect() {
        historySelect.innerHTML = '<option value="">-- 选择历史记录 --</option>';
        
        clipboardHistory.forEach((item, index) => {
            const option = document.createElement('option');
            option.value = index;
            
            // 截断长文本用于显示
            const displayText = item.length > 30 ? 
                item.substring(0, 30) + '...' : item;
            option.textContent = displayText;
            option.title = item; // 鼠标悬停显示完整内容
            
            historySelect.appendChild(option);
        });
    }

    // 保存到历史记录
    function saveToHistory(text) {
        if (!text || text.trim() === '') return;
        
        // 移除重复项
        clipboardHistory = clipboardHistory.filter(item => item !== text);
        
        // 添加到开头
        clipboardHistory.unshift(text);
        
        // 限制历史记录数量
        if (clipboardHistory.length > MAX_HISTORY_ITEMS) {
            clipboardHistory = clipboardHistory.slice(0, MAX_HISTORY_ITEMS);
        }
        
        // 保存到存储
        try {
            GM_setValue('clipboardHistory', clipboardHistory);
        } catch (e) {
            console.error('保存历史记录失败:', e);
        }
        
        updateHistorySelect();
    }

    // 读取剪切板内容
    async function readClipboard() {
        try {
            readClipboardButton.disabled = true;
            readClipboardButton.textContent = '读取中...';
            
            let newText = '';
            
            // 尝试使用现代 Clipboard API
            if (navigator.clipboard && navigator.clipboard.readText) {
                newText = await navigator.clipboard.readText();
            } 
            // 尝试使用 Tampermonkey API
            else if (typeof GM_getClipboard === 'function') {
                newText = await new Promise((resolve) => {
                    GM_getClipboard(resolve);
                });
            }
            // 备用方案
            else {
                const tempTextarea = document.createElement('textarea');
                tempTextarea.style.position = 'fixed';
                tempTextarea.style.opacity = '0';
                document.body.appendChild(tempTextarea);
                tempTextarea.focus();
                
                const success = document.execCommand('paste');
                if (success) {
                    newText = tempTextarea.value;
                } else {
                    throw new Error('无法访问剪切板');
                }
                
                document.body.removeChild(tempTextarea);
            }

            if (newText && newText !== clipboardText) {
                clipboardText = newText;
                clipboardPreview.value = clipboardText.length > 100 ? 
                    clipboardText.substring(0, 100) + '...' : clipboardText;
                clipboardPreview.title = clipboardText;
                
                // 保存到历史记录
                saveToHistory(clipboardText);
                
                statusText.textContent = '读取成功！';
                setTimeout(() => {
                    if (!isTyping) statusText.textContent = '';
                }, 2000);
            } else if (!newText) {
                statusText.textContent = '剪切板为空';
                setTimeout(() => {
                    if (!isTyping) statusText.textContent = '';
                }, 2000);
            }
        } catch (error) {
            console.error('读取剪切板失败:', error);
            statusText.textContent = '读取失败，请重试';
            setTimeout(() => {
                if (!isTyping) statusText.textContent = '';
            }, 2000);
        } finally {
            readClipboardButton.disabled = false;
            readClipboardButton.textContent = '读取剪切板';
        }
    }

    // 自动刷新剪切板
    function toggleAutoRefresh() {
        if (refreshCheckbox.checked) {
            const interval = parseInt(refreshInput.value) * 1000;
            if (interval < 1000) {
                statusText.textContent = '刷新间隔太短';
                setTimeout(() => {
                    if (!isTyping) statusText.textContent = '';
                }, 2000);
                refreshCheckbox.checked = false;
                return;
            }
            
            autoRefreshInterval = setInterval(() => {
                if (!isTyping) {
                    readClipboard();
                }
            }, interval);
            
            statusText.textContent = '自动刷新已开启';
            setTimeout(() => {
                if (!isTyping) statusText.textContent = '';
            }, 2000);
        } else {
            if (autoRefreshInterval) {
                clearInterval(autoRefreshInterval);
                autoRefreshInterval = null;
            }
            statusText.textContent = '自动刷新已关闭';
            setTimeout(() => {
                if (!isTyping) statusText.textContent = '';
            }, 2000);
        }
    }

    // 获取延迟时间
    function getRandomDelay() {
        const speed = speedSelect.value;
        let minDelay, maxDelay;

        switch(speed) {
            case 'ultra-fast':
                minDelay = 1;
                maxDelay = 5;
                break;
            case 'fast':
                minDelay = 5;
                maxDelay = 20;
                break;
            case 'slow':
                minDelay = 70;
                maxDelay = 150;
                break;
            case 'medium':
            default:
                minDelay = 30;
                maxDelay = 70;
                break;
        }

        return minDelay + Math.random() * (maxDelay - minDelay);
    }

    function getThinkingDelay() {
        const speed = speedSelect.value;

        switch(speed) {
            case 'ultra-fast':
                return 0; // 超高速不需要思考停顿
            case 'fast':
                return 30 + Math.random() * 60;
            case 'slow':
                return 200 + Math.random() * 200;
            case 'medium':
            default:
                return 50 + Math.random() * 100;
        }
    }

    // 触发编辑器事件
    function triggerEditorEvents(element) {
        const inputEvent = new Event('input', { bubbles: true, cancelable: true });
        element.dispatchEvent(inputEvent);

        const changeEvent = new Event('change', { bubbles: true, cancelable: true });
        element.dispatchEvent(changeEvent);

        element.dispatchEvent(new Event('focus', { bubbles: true }));
        element.dispatchEvent(new Event('blur', { bubbles: true }));
        element.dispatchEvent(new Event('focus', { bubbles: true }));
    }

    // 插入字符
    function insertCharAndTriggerEvents(targetElement, char) {
        if (targetElement.isContentEditable) {
            document.execCommand('insertText', false, char);
        } else if (targetElement.tagName === 'TEXTAREA' || targetElement.tagName === 'INPUT') {
            targetElement.value += char;
        } else {
            targetElement.textContent += char;
        }

        triggerEditorEvents(targetElement);
    }

    // 模拟人类打字
    async function simulateHumanTyping(text, targetElement, mode) {
        isTyping = true;
        cancelTyping = false;
        pasteButton.disabled = true;
        readClipboardButton.disabled = true;
        clearButton.disabled = true;
        modeSelect.disabled = true;
        speedSelect.disabled = true;
        historySelect.disabled = true;
        refreshCheckbox.disabled = true;
        refreshInput.disabled = true;
        pasteButton.textContent = '粘贴中...';
        progressContainer.style.display = 'flex';
        statusText.textContent = '准备开始输入...';

        targetElement.style.setProperty('white-space', 'pre-wrap', 'important');
        targetElement.style.setProperty('font-family', 'monospace', 'important');
        targetElement.style.setProperty('line-height', '1.5', 'important');

        if (mode === 'replace') {
            if (targetElement.isContentEditable) {
                targetElement.textContent = '';
            } else if (targetElement.tagName === 'TEXTAREA' || targetElement.tagName === 'INPUT') {
                targetElement.value = '';
            } else {
                targetElement.textContent = '';
            }
            triggerEditorEvents(targetElement);
        }

        const totalChars = text.length;
        let typedChars = 0;

        for (let i = 0; i < totalChars; i++) {
            if (cancelTyping) {
                break;
            }

            const char = text[i];
            typedChars++;
            const progress = (typedChars / totalChars) * 100;
            statusText.textContent = `输入中... ${Math.round(progress)}%`;

            insertCharAndTriggerEvents(targetElement, char);

            await new Promise(resolve => {
                setTimeout(resolve, getRandomDelay());
            });

            // 超高速模式下不需要思考停顿
            if (speedSelect.value !== 'ultra-fast' && i % 20 === 19 && i < totalChars - 1) {
                await new Promise(resolve => {
                    setTimeout(resolve, getThinkingDelay());
                });
            }
        }

        isTyping = false;
        pasteButton.disabled = false;
        readClipboardButton.disabled = false;
        clearButton.disabled = false;
        modeSelect.disabled = false;
        speedSelect.disabled = false;
        historySelect.disabled = false;
        refreshCheckbox.disabled = false;
        refreshInput.disabled = false;
        pasteButton.textContent = '开始粘贴';
        progressContainer.style.display = 'none';

        if (cancelTyping) {
            statusText.textContent = '输入已取消';
            setTimeout(() => {
                statusText.textContent = '';
            }, 2000);
        } else {
            statusText.textContent = '输入完成！';
            triggerEditorEvents(targetElement);
            setTimeout(() => {
                statusText.textContent = '';
            }, 2000);
        }
    }

    // 取消输入
    function cancelTypingProcess() {
        if (isTyping) {
            cancelTyping = true;
            pasteButton.textContent = '取消中...';
            statusText.textContent = '正在取消...';
        }
    }

    // 清空内容
    clearButton.addEventListener('click', () => {
        clipboardText = '';
        clipboardPreview.value = '';
        clipboardPreview.title = '';
        historySelect.value = '';
    });

    // 历史记录选择事件
    historySelect.addEventListener('change', () => {
        const selectedIndex = parseInt(historySelect.value);
        if (!isNaN(selectedIndex) && selectedIndex >= 0 && selectedIndex < clipboardHistory.length) {
            clipboardText = clipboardHistory[selectedIndex];
            clipboardPreview.value = clipboardText.length > 100 ? 
                clipboardText.substring(0, 100) + '...' : clipboardText;
            clipboardPreview.title = clipboardText;
        }
    });

    // 读取剪切板按钮事件
    readClipboardButton.addEventListener('click', readClipboard);

    // 自动刷新设置事件
    refreshCheckbox.addEventListener('change', toggleAutoRefresh);
    refreshInput.addEventListener('change', () => {
        if (refreshCheckbox.checked) {
            // 如果自动刷新已开启，更新间隔
            toggleAutoRefresh();
            toggleAutoRefresh();
        }
    });

    // 取消按钮事件
    cancelButton.addEventListener('click', cancelTypingProcess);

    // 粘贴按钮事件
    pasteButton.addEventListener('click', () => {
        if (!clipboardText) {
            statusText.textContent = '请先读取剪切板或选择历史记录';
            setTimeout(() => {
                statusText.textContent = '';
            }, 2000);
            return;
        }

        if (isTyping) {
            cancelTypingProcess();
            return;
        }

        const contentDiv = document.querySelector('div.cm-content, div[contenteditable="true"]')
            || document.querySelector('div[contenteditable]')
            || document.querySelector('div.CodeMirror-code');

        if (!contentDiv) {
            alert('未找到内容区域，请确保在题目页面。');
            return;
        }

        const mode = modeSelect.value;
        simulateHumanTyping(clipboardText, contentDiv, mode);
    });

    // 初始化
    initHistory();
    
    // 自动尝试读取剪切板
    setTimeout(() => {
        readClipboard();
    }, 1000);

    // 确保脚本在页面加载后执行
    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', () => {
            document.body.appendChild(container);
        });
    }
})();
