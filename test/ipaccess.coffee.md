    mocha = require 'mocha'
    should = require 'should'

    {EventEmitter} = require 'events'

    describe 'When using IP.Access', ->
      ipaccessProtocol = (require '..').protocol
      ipaccessHandler = (require '..').handler
      describe 'the protocol handler', ->
        it 'should respond with pong to a ping', (done) ->
          conn = new EventEmitter
          ipaccess = new ipaccessProtocol conn
          handler = new ipaccessHandler ipaccess
          conn.send = (proto,frame) ->
            proto.should.equal 'ip_access'
            frame.should.have.length 1
            frame.toJSON().should.eql [0x01]
            done()
          conn.emit 'protocol ip_access', new Buffer [ 0x00 ]
