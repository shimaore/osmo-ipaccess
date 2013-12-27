IP Access protocol adaptation layer
===================================

    {EventEmitter} = require 'events'

    module.exports = class ipAccessProtocol extends EventEmitter

      constructor: (@connection) ->
        @init()
        @connection.on 'protocol ip_access', (data) =>
          @parse data

Send
----

      send: (command,params,cb) ->
        # params is optional
        if typeof params is 'function'
          [params,cb] = [null,params]
        if not params?
          params = new Buffer 0
        # /optional

        frame = new Buffer params.length + 1
        if typeof command is 'string'
          command = @msgtype[command]
        frame.writeUInt8 command, 0
        params.copy frame, 1
        @connection.send 'ip_access', frame, cb

Parser
------

The parser handles incoming data segments, parses them and takes appropriate action.

      parse: (data) ->

        if data.length < 1
          console.error "IP Access parser: data is too short"
          return

IP.Access messages starts with a command designator, then optional data.

        command_id = data.readUInt8 0
        command = @msgname[command_id]

        if command?
          params = data.slice 1
          @emit command, params
        else
          console.error "IP Access parser: unknown command #{command_id}"


Message Types
-------------

      msgtype:
        ping:     0x00
        pong:     0x01
        id_get:   0x04
        id_resp:  0x05
        id_ack:   0x06
        # OpenBSC extension
        sccp_old: 0xff

      init: ->
        @msgname = {}
        for name, id of @msgtype
          @msgname[id] = name
