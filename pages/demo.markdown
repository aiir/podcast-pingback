# Proof of Concept

The following is a standard HTML5 audio element. Attached is a Javascript-based
listener observing playback events and simulating the reporting that would take
place to comply with [the specification](/specification/1).

As events occur, the simulated HTTP requests appear below the player, with the
most recent at the top of the list.

This code is not currently considered production ready, but it is licensed under
the MIT License, should you wish to peer behind the curtain and view the source.

<script>
  /**
   * Podcast Pingback Proof-of-concept HTML5 Audio element "listener"
   *
   * MIT License
   *
   * Copyright (c) 2018 Aiir
   *
   * Permission is hereby granted, free of charge, to any person obtaining a copy
   * of this software and associated documentation files (the "Software"), to deal
   * in the Software without restriction, including without limitation the rights
   * to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
   * copies of the Software, and to permit persons to whom the Software is
   * furnished to do so, subject to the following conditions:
   *
   * The above copyright notice and this permission notice shall be included in
   * all copies or substantial portions of the Software.
   *
   * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
   * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
   * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
   * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
   * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
   * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
   * SOFTWARE.
   */

  function podcastPingback(options) {
    if (options === undefined) {
      options = {};
    }

    var overrideReceiverUrl = options.overrideReceiverUrl;
    var receiverUrlDataAttributeKey = options.receiverUrlDataAttributeKey || 'podcastPingbackReceiverUrl';
    var debugCallback = options.debugCallback;

    var uuid;

    var lastSuspendEvent;

    var eventQueue = [];
    var lastEventSent;
    var nextEventDispatchTimeout;

    function generateUUID() {
      var uuid = '';
      for (var i = 0; i < 32; i++) {
        var random = Math.random() * 16 | 0;
        if (i === 8 || i === 12 || i === 16 || i === 20) {
          uuid += '-'
        }
        uuid += (i === 12 ? 4 : (i === 16 ? (random & 3 | 8) : random)).toString(16);
      }
      return uuid;
    }

    function dispatchEvent(receiverUrl, content, event, offset, options) {
      if (options === undefined) {
        options = {};
      }
      var formattedEvent = {
        event,
        date: options.date || (new Date()).toISOString(),
        offset,
      };
      var keys = Object.keys(options);
      for (var i = 0; i < keys.length; i++) {
        var key = keys[i];
        if (
          key.toLowerCase() === 'server' ||
          key.toLowerCase() === 'event' ||
          key.toLowerCase() === 'date' ||
          key.toLowerCase() === 'offset'
        ) {
          continue;
        }
        formattedEvent[key] = options[key];
      }
      var body = {
        content,
        uuid,
        events: [
          formattedEvent,
        ],
      };
      // TODO: actually send the event here
      if (typeof debugCallback === 'function') {
        debugCallback(receiverUrl, body);
      }
    }

    function clearLastSuspendEvent() {
      if (lastSuspendEvent === undefined) {
        return;
      }
      lastSuspendEvent = undefined;
    }

    function dispatchLastSuspendEvent() {
      if (lastSuspendEvent === undefined) {
        return;
      }
      var receiverUrl = lastSuspendEvent.receiverUrl;
      var content = lastSuspendEvent.content;
      var offset = lastSuspendEvent.event.offset;
      var options = lastSuspendEvent.event;
      dispatchEvent(receiverUrl, content, 'suspend', offset, options);
      clearLastSuspendEvent();
    }

    function handlePlayEvent() {
      dispatchLastSuspendEvent();
      var receiverUrl = overrideReceiverUrl;
      if (receiverUrl === undefined && this.dataset !== undefined) {
        receiverUrl = this.dataset[receiverUrlDataAttributeKey];
      }
      var content = this.src;
      var offset = this.currentTime;
      dispatchEvent(receiverUrl, content, 'resume', offset);
    }

    function handlePauseEvent() {
      var receiverUrl = overrideReceiverUrl;
      if (receiverUrl === undefined && this.dataset !== undefined) {
        receiverUrl = this.dataset[receiverUrlDataAttributeKey];
      }
      lastSuspendEvent = {
        receiverUrl,
        content: this.src,
        event: {
          reason: 'pause',
          offset: this.currentTime,
        },
      };
    }

    function handleSeekEvent() {
      if (lastSuspendEvent === undefined || lastSuspendEvent.reason === 'seek') {
        return;
      }
      var receiverUrl = overrideReceiverUrl;
      if (receiverUrl === undefined && this.dataset !== undefined) {
        receiverUrl = this.dataset[receiverUrlDataAttributeKey];
      }
      lastSuspendEvent = {
        receiverUrl,
        content: this.src,
        event: {
          reason: 'seek',
          offset: this.currentTime,
        },
      };
    }

    function handleEndEvent() {
      clearLastSuspendEvent();
      var receiverUrl;
      if (this.dataset !== undefined) {
        receiverUrl = this.dataset[receiverUrlDataAttributeKey] || overrideReceiverUrl;
      }
      var content = this.src;
      var offset = this.currentTime;
      dispatchEvent(receiverUrl, content, 'suspend', offset, { reason: 'complete' });
    }

    if (options.uuid) {
      uuid = options.uuid;
    } else {
      var localStorageKey = options.localStorageKey || 'podcast-pingback-uuid';
      if (window.localStorage !== undefined) {
        uuid = localStorage.getItem(localStorageKey);
      }
      if (uuid === null) {
        uuid = generateUUID();
        if (window.localStorage !== undefined) {
          localStorage.setItem(localStorageKey, uuid);
        }
      }
    }

    var players;
    if (options.players !== undefined) {
      players = Array.isArray(options.players) ? options.players : [options.players];
    } else {
      var className = options.className || 'podcast-pingback-player';
      players = document.getElementsByClassName(className);
    }

    for (var i = 0; i < players.length; i++) {
      var player = players[i];

      var receiverUrl = overrideReceiverUrl;
      if (receiverUrl === undefined && player.dataset !== undefined) {
        receiverUrl = player.dataset[receiverUrlDataAttributeKey];
        if (receiverUrl === undefined) {
          var error = new Error('No pingback receiver data attribute or override option set');
          throw error;
        }
      }
      if (receiverUrl.substring(0, 6) !== 'https:') {
        var error = new Error('Pingback receiver must be HTTPS');
        throw error;
      }

      player.addEventListener('play', handlePlayEvent);
      player.addEventListener('pause', handlePauseEvent);
      player.addEventListener('seeking', handleSeekEvent);
      player.addEventListener('ended', handleEndEvent);
      player.addEventListener('mouseout', dispatchLastSuspendEvent);
    }
  }
</script>
<audio class="podcast-pingback-player" src="https://www.aiir.com/podcasts/aiir-podcast/1.mp3" data-podcast-pingback-receiver-url="https://example/receiver" controls style="width:100%;">
  <p>Sorry, the audio element is not supported in this browser.</p>
</audio>
<ul id="log" style="padding:0;"></ul>
<script>
  var log = document.getElementById('log');
  podcastPingback({
    // uuid: 'ff934b94-3651-4935-901d-8eac8eb311dc',
    // localStorageKey: 'podcast-pingback-uuid',
    // players: document.getElementById('player'),
    // className: 'podcast-pingback-player',
    // overrideReceiverUrl: 'http://host/receiver',
    // receiverUrlDataAttributeKey: 'podcast-pingback-receiver-url',
    debugCallback: function(receiverUrl, event) {
      var item = document.createElement('li');
      var url = new URL(receiverUrl);
      var json = JSON.stringify(event, null, '  ');
      item.innerHTML = '<pre>POST ' + url.pathname + '\nHost: ' + url.hostname + '\nContent-Length: ' + json.length + '\n\n' + json + '</pre>';
      log.insertBefore(item, log.firstChild);
    },
  });
</script>
